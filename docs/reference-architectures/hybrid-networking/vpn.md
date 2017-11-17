---
title: "Egy helyszíni hálózat csatlakozik az Azure VPN-nel"
description: "Hogyan egy biztonságos pont-pont hálózati architektúra, amely egy Azure virtuális hálózatra és egy a helyszíni hálózathoz egy VPN-nel csatlakoztatott kiterjedő végrehajtásához."
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: 66b2605c551148fadcdee6808c4e85940089f1e5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Egy helyszíni hálózat csatlakozik az Azure VPN-átjáró használatával

A referencia-architektúrában bemutatja, hogyan egy a helyszíni hálózat kiterjesztése az Azure-ba, pont-pont virtuális magánhálózat (VPN) segítségével. A forgalom között a helyszíni hálózat és az Azure Virtual Network (VNet), az IPSec VPN-alagúton keresztül. [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra 

Az architektúra a következő összetevőkből áll.

* **A helyszíni hálózat**. Helyi magánhálózat fut egy szervezeten belül.

* **VPN-készülék**. Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás. Lehet, hogy a VPN-készülék olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben. Támogatott VPN-készülék és azok tudjanak csatlakozni az Azure VPN gatewayhez konfigurálásáról listájáért lásd: a kijelölt eszköz-cikkben található utasításokat [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok] [ vpn-appliance].

* **Virtuális hálózathoz (VNet)**. A felhőalapú alkalmazások és az Azure VPN gateway a összetevőket található azonos [VNet][azure-virtual-network].

* **Az Azure VPN gateway**. A [VPN-átjáró] [ azure-vpn-gateway] szolgáltatás lehetővé teszi, hogy a virtuális hálózat csatlakozni a helyszíni hálózathoz egy VPN-készülék keresztül. További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet]. A VPN-átjáró a következő elemeket tartalmazza:
  
  * **Virtuális hálózati átjáró**. Egy virtuális VPN-készülék biztosít a virtuális hálózat erőforrás. Ez felelős adatforgalom a helyi hálózatról a virtuális hálózatba.
  * **Helyi hálózati átjáró**. A helyszíni VPN-készülék absztrakciós. A felhő alkalmazásról, a helyszíni hálózat a hálózati forgalom az átjáró keresztül történik.
  * **Kapcsolat**. A kapcsolat tulajdonságai adja meg a kapcsolat típusát (IPSec) és a kulcsot, a helyszíni VPN-készülék megosztott forgalom titkosításához.
  * **Átjáró alhálózati**. A virtuális hálózati átjáró saját alhálózatba, amely különböző vonatkoznak, az ajánlások szakaszban leírt használatban van.

* **A felhő alkalmazás**. Az alkalmazást az Azure-ban. A többrétegű konfigurációk – Azure load Balancer terheléselosztók keresztül kapcsolódik, több alhálózattal rendelkező tartalmazhat. Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].

* **Belső terheléselosztó**. Hálózati forgalmat a VPN-átjárót a felhőalapú alkalmazásokhoz, a belső terheléselosztók keresztül van átirányítva. A terheléselosztó előtér-alkalmazás az alhálózaton található.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="vnet-and-gateway-subnet"></a>Virtuális hálózat és az átjáró-alhálózatot

Hozzon létre egy Azure virtuális hálózatot egy cím elegendő hely számára összes szükséges erőforrást. Győződjön meg arról, hogy a virtuális hálózat címtér rendelkezik-e elegendő hely marad, ha további virtuális gépek valószínűleg a jövőben van szükség. A virtuális hálózat címtartománya nem fedhetik a helyszíni hálózattal. Például a fenti ábra a cím terület 10.20.0.0/16 használ a virtuális hálózat.

Hozzon létre egy nevű alhálózatot *GatewaySubnet*, a /27 címtartományt. Ez az alhálózat által a virtuális hálózati átjáró szükséges. Lefoglalásával megelőzése érdekében az alhálózathoz 32 cím segítségével átjáró méretkorlátai a jövőben elérése. Emellett helyez az alhálózat a címterület közepén. Bevált gyakorlat is, hogy az átjáró alhálózatának a címtartomány a virtuális hálózat címterület felső végén. Az ábrán is látható példa 10.20.255.224/27.  Íme egy gyors eljárás kiszámításához a [CIDR]:

1. Állítsa be a változó bitek a címterületen belülre a vnet, akár a bits alatt az átjáró-alhálózat használja, akkor a többi bit értéke 0, 1.
2. Az eredményül kapott bits átalakítása decimális, és azt, az átjáró alhálózatának méretét a előtag hossza értéke egy címtartománnyal express.

Például egy IP-címtartománnyal 10.20.0.0/16 a vnet, alkalmazza a fenti #1. lépés lesz 10.20.0b11111111.0b11100000.  Konvertálása, decimális és kifejező azt, egy 10.20.255.224/27 adja eredményül. 

> [!WARNING]
> Nem kell telepítenie a virtuális gépek az átjáró alhálózatához. Is nem rendel egy NSG-t az alhálózaton, mint okozhat az átjáró működését.
> 
> 

### <a name="virtual-network-gateway"></a>Virtuális hálózati átjáró

A virtuális hálózati átjáró nyilvános IP-címnek lefoglalni.

Hozza létre a virtuális hálózati átjárót, az átjáró-alhálózatot, és rendelje hozzá az újonnan lefoglalt nyilvános IP-cím. Nyomja meg az átjáró típusa, amely a leginkább megfelel a követelményeknek, és a VPN-készülék szerint engedélyezve van, amely:

- Hozzon létre egy [átjáró házirendalapú] [ policy-based-routing] Ha kell szigorúan szabályozzák a rendszer kérést átirányítja milyen címelőtagokat házirend feltételek alapján. Csoportházirend-alapú átjárók használható statikus útválasztás, és csak a pont-pont kapcsolatok működnek.

- Hozzon létre egy [útválasztó-alapú átjáró] [ route-based-routing] csatlakozhat a helyszíni hálózat, az RRAS használata, ha támogatja a többhelyes vagy kereszt-terület kapcsolatokat, vagy valósítja meg a VNet – VNet kapcsolatokhoz (beleértve a továbbítja, amely haladnak át több Vnetekről). Útválasztó-alapú átjárók használja, a hálózatok közötti közvetlen forgalom dinamikus útválasztást. Azok tűri hibák jobb, mint a statikus útvonal, hálózati elérési útról, mert azok próbálkozhat alternatív útvonalak. Útválasztó-alapú átjárók is csökkentheti a felügyeleti mert útvonalak előfordulhat, hogy nem kell manuálisan frissül, amikor megváltozik hálózati címe.

Támogatott VPN-készülékek listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliances].

> [!NOTE]
> Az átjáró létrehozása után nem módosítható nélkül törli, majd újra létrehozza az átjáró átjáró típusok között.
> 
> 

Válassza ki az Azure VPN gateway SKU, amely a leginkább megfelel a átviteli követelményeknek. Az Azure VPN-átjáró a következő táblázatban látható három termékváltozatok érhető el. 

| SKU | VPN átviteli sebesség | Maximális IPSec-alagutak |
| --- | --- | --- |
| Basic |100 Mbps |10 |
| Standard |100 Mbps |10 |
| Kiemelkedő teljesítmény |200 Mbps |30 |

> [!NOTE]
> Az alapvető termékváltozata nem kompatibilis a Azure ExpressRoute. Is [módosítása a Termékváltozat] [ changing-SKUs] az átjáró létrehozása után.
> 
> 

Az, hogy mennyi ideig az átjáró kiépítését alapján, és elérhető van szó. Lásd: [VPN-átjáró árképzési][azure-gateway-charges].

Hozzon létre az átjáró alhálózatának vonatkozó útválasztási szabályokat, hogy közvetlen bejövő alkalmazás forgalom a belső terheléselosztó ahelyett, hogy közvetlenül az alkalmazás virtuális gépek átadása kérésének engedélyezése az átjárót.

### <a name="on-premises-network-connection"></a>A helyszíni hálózati kapcsolat

A helyi hálózati átjáró létrehozása. Adja meg a helyszíni VPN-készülék, és a helyszíni hálózat a címtér nyilvános IP-címét. Vegye figyelembe, hogy a helyszíni VPN-készülék rendelkeznie kell egy nyilvános IP-címet, amely hozzáfér a helyi hálózati átjáró az Azure VPN Gateway. A VPN-eszköz mögött egy hálózati címfordítási (NAT) eszköz nem található.

A virtuális hálózati átjáró és a helyi hálózati átjáró webhelyek kapcsolatot létrehozni. Válassza ki a pont-pont (IPSec) kapcsolattípus, és adja meg a megosztott kulcsot. Az Azure VPN gateway-webhelyek titkosítás az IPSec protokoll használatával az előmegosztott kulcsos hitelesítés alapul. Adja meg a kulcsot az Azure VPN gateway létrehozásakor. A VPN-készülék helyileg futó ugyanazzal a kulccsal kell konfigurálnia. Más hitelesítési mechanizmusok jelenleg nem támogatottak.

Győződjön meg arról, hogy a helyi útválasztási infrastruktúra úgy van beállítva, a VPN-eszköz az Azure VNet címek szánt kérelmek továbbítása.

Nyissa meg a felhő alkalmazáshoz, a helyszíni hálózat szükséges portja.

Ellenőrizze, hogy a kapcsolat ellenőrzéséhez:

* A helyszíni VPN-készülék megfelelően irányítja a forgalmat a felhő-alkalmazás az Azure VPN-átjárón keresztül.
* A virtuális hálózat megfelelően irányítja a forgalmat vissza a helyszíni hálózat számára.
* Mindkét irányban tiltott forgalmat megfelelően le van tiltva.

## <a name="scalability-considerations"></a>Méretezési szempontok

Korlátozott függőleges méretezhetőség érhet el, a Basic vagy Standard VPN Gateway SKU-n áthelyezését a magas teljesítmény VPN Termékváltozat.

VPN-forgalmát nagy mennyiségű teheti Vnetek fontolja meg a különböző terhelésekhez külön kisebb Vnetek történő terjesztése, és azok a VPN-átjáró konfigurálása.

A VNet is particionálása, vízszintes vagy függőleges irányú. Vízszintes particionálása, áthelyezni egyes Virtuálisgép-példányok egyes rétegek az új VNet alhálózatainak. Az eredménye, hogy minden virtuális hálózat rendelkezik-e ugyanazokat a struktúra és funkciókat. Függőleges particionálásához, Újratervezés az egyes rétegek segítségével kiszámolja a funkciók különböző logikai területekre (például rendelések kezelése, a számlázással, felhasználói fiókok kezelése és így tovább). Minden egyes területhez majd a saját virtuális kell elhelyezni.

Egy a helyszíni Active Directory-tartományvezérlőt a VNet replikálásának és valósít meg a virtuális hálózat DNS segíthet a helyszíni áramlanak a felhőbe biztonsági és felügyeleti forgalom bizonyos részének csökkentése érdekében. További információkért lásd: [kiterjesztése az Active Directory tartományi szolgáltatások (AD DS) az Azure-bA][adds-extend-domain].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Győződjön meg arról, hogy a helyszíni hálózat továbbra is elérhető az Azure VPN gatewayhez van szüksége, ha valósítja meg a helyszíni VPN-átjáró a feladatátvevő fürtökben.

Ha a szervezete több helyszíni helyeken, hozzon létre [többhelyes kapcsolatok] [ vpn-gateway-multi-site] egy vagy több Azure Vnet számára. Ez a megközelítés megadását követeli meg dinamikus (útválasztó-alapú), ezért győződjön meg arról, hogy a helyszíni VPN-átjáró támogatja-e ezt a szolgáltatást.

További információk a szolgáltatásszint-szerződések: [SLA-t, a VPN-átjáró][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

A helyszíni VPN készülékek diagnosztikai információk figyelését. Ez az eljárás attól függ, hogy a VPN-készülék által nyújtott szolgáltatásokat. Ha használja az Útválasztás és távelérés szolgáltatás Windows Server 2012-ben például [RRAS naplózási][rras-logging].

Használjon [Azure VPN gateway diagnosztika] [ gateway-diagnostic-logs] kapcsolódási problémák vonatkozó információkat. Ezek a naplók információkat, például a forrás és a célhelyek csatlakozási kérelmek, használt protokollt, és hogyan jött létre a kapcsolat (vagy miért nem sikerült) nyomon követésére használható.

Figyeléséhez használja a felügyeleti naplók az Azure portálon elérhető Azure VPN-átjáró a műveleti naplókat. Külön elérhetők naplók a helyi hálózati átjáró, az Azure-hálózatot átjáró és a kapcsolat. Ezt az információt az átjáró végrehajtott módosítások követésére használható, és akkor lehet hasznos, ha korábban már működő átjáró valamilyen okból kifolyólag nem működik.

![[2]][2]

Figyelje a kapcsolatot, és kövesse a kapcsolatra vonatkozó hibaesemények. Használhatja például a felügyeleti csomag [Nagios] [ nagios] rögzítéséhez, és megjeleníti ezt.

## <a name="security-considerations"></a>Biztonsági szempontok

Minden egyes VPN-átjáró különböző megosztott kulcs létrehozása. Megosztott kulcs erős a segítségével ellenáll a találgatásos támadások során.

> [!NOTE]
> Az Azure Key Vault jelenleg nem használható az Azure VPN gateway kulcsokat preshare.
> 
> 

Győződjön meg arról, hogy a helyszíni VPN-készülék használ, amely titkosítási metódus [kompatibilis az Azure VPN gateway][vpn-appliance-ipsec]. A házirendalapú útválasztást, az Azure VPN gateway AES256 AES128 és a 3DES titkosítási algoritmus támogatja. Útválasztó-alapú átjárók AES256 és 3DES támogatja.

Ha a helyszíni VPN-készülék szegélyhálózaton (DMZ) van-e a peremhálózat és az Internet között tűzfal található, lehetséges, hogy konfigurálása [további tűzfalszabályok] [ additional-firewall-rules] engedélyezi a pont-pont VPN-kapcsolatot.

Ha az alkalmazás virtuális adatokat küld az interneten, fontolja meg a [végrehajtási kényszerített bújtatás] [ forced-tunneling] irányíthatja az összes internetes-forgalmat a helyi hálózaton keresztül. Ez a megközelítés lehetővé teszi naplózása által az alkalmazás a helyszíni infrastruktúra kimenő kérelmek.

> [!NOTE]
> A kényszerített bújtatás hatással lehet az Azure-szolgáltatások (a tároló szolgáltatás, például) és a Windows Licenckezelő való kapcsolódást.
> 
> 


## <a name="troubleshooting"></a>Hibaelhárítás 

VPN-kapcsolatos gyakori hibák elhárításával kapcsolatos általános információkért lásd: [közös VPN hibaelhárítási kapcsolódó hibák][troubleshooting-vpn-errors].

A következő javaslatok még meghatározásához hasznos, ha a helyszíni VPN-készülék megfelelően működik-e.

- **A VPN-készülék a hibákat, a hibák által létrehozott naplófájlok ellenőrzése.**

    Ez segít meghatározni, ha a VPN-készülék megfelelően működik-e. Ezek az információk helyét a készülék függően változnak. Például RRAS használ a Windows Server 2012, a következő PowerShell-parancsot a hiba esemény megjelenítheti az RRAS szolgáltatás is használhatja:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    A *üzenet* az egyes bejegyzések tulajdonság tartalmazza a hiba leírása. Néhány gyakori példák:

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
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

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
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

    Eseménynapló információ kísérel meg csatlakozni az RRAS szolgáltatás, a következő PowerShell-paranccsal is szerezheti be: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    Csatlakozás meghibásodása ezt a naplót az alábbihoz hasonló hibák fogja tartalmazni:

    ```
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

- **Ellenőrizze a kapcsolatot és továbbítását a VPN-átjáró között.**

    A VPN-készülék előfordulhat, hogy nem lehet megfelelően útválasztási az Azure VPN Gateway forgalmát. Használjon például egy eszköz [PsPing] [ psping] ellenőrizni a kapcsolatot és továbbítását a VPN-átjáró között. Például egy olyan kiszolgálóra, a virtuális hálózaton található a helyi gépről kapcsolat tesztelése, a következő parancsot (cseréje `<<web-server-address>>` a webalkalmazás-kiszolgáló címe):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Ha a helyszíni gépre irányíthatja a forgalmat a webkiszolgálónak, a következőhöz hasonló kimenetnek kell megjelennie:

    ```
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

    A helyi számítógép nem tud kommunikálni a megadott helyre, ha ehhez hasonló üzenetet fog látni:

    ```
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

- **Győződjön meg arról, hogy a helyi tűzfal engedélyezi-e a VPN-adatforgalmat, és a megfelelő portokat.**

- **Győződjön meg arról, hogy a helyszíni VPN-készülék használ, amely titkosítási metódus [kompatibilis az Azure VPN gateway][vpn-appliance].** A házirendalapú útválasztást, az Azure VPN gateway AES256 AES128 és a 3DES titkosítási algoritmus támogatja. Útválasztó-alapú átjárók AES256 és 3DES támogatja.

Az alábbi javaslatokat hasznosak meghatározhatja, hogy az Azure VPN gateway probléma:

- **Vizsgálja meg [Azure VPN-átjáró diagnosztikai naplók] [ gateway-diagnostic-logs] az esetleges problémákat.**

- **Győződjön meg arról, hogy ugyanazt a megosztott hitelesítési kulcsot az Azure VPN gateway és a helyszíni VPN-készülék vannak konfigurálva.**

    A megosztott kulcs tárolja az Azure VPN gateway a következő Azure CLI-parancs használatával tekintheti meg:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    A paranccsal a helyszíni VPN-készülék megfelelő megjelenítése a megosztott kulcs van konfigurálva, hogy alapplatformjaként.

    Ellenőrizze, hogy a *GatewaySubnet* alhálózat az Azure VPN gateway okozó nincs társítva egy NSG.

    Az alhálózati adatokat, a következő Azure CLI-parancs használatával tekintheti meg:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Gondoskodjon arról, hogy nincs nevű adatmező *hálózati biztonsági csoport azonosítója*. A következő példa egy példányát az eredményeit jeleníti meg a *GatewaySubnet* , amely rendelkezik hozzárendelt NSG-t (*VPN-átjáró-csoport*). Emiatt előfordulhat, hogy megfelelően működik, ha bármely szabály lett meghatározva az NSG az átjárót.

    ```
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

- **Győződjön meg arról, hogy a virtuális gépeket az Azure virtuális hálózatot a forgalmat a érkező a virtuális hálózaton kívül van konfigurálva.**

    Bármely, a virtuális gépeket tartalmazó alhálózattal társított NSG-szabályok keresése Minden NSG-szabályok a következő Azure CLI-parancs használatával tekintheti meg:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Győződjön meg arról, hogy csatlakozik-e az Azure VPN gateway.**

    A következő Azure PowerShell-parancs segítségével az Azure VPN-kapcsolat jelenlegi állapotának ellenőrzése. A `<<connection-name>>` paramétere az Azure VPN-kapcsolat, amely a virtuális hálózati átjáró és a helyi átjáró nevét.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    Az alábbi kódtöredékek jelölje ki a kimenetet állít elő, ha az átjáró csatlakoztatva (az első példában), és leválasztott (a második példában):

    ```
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

    ```
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

Az alábbi javaslatokat hasznosak meghatározhatja, hogy a gazdagép VM konfigurációs, a hálózati sávszélesség-használatot vagy az alkalmazások teljesítményének problémát:

- **Győződjön meg arról, hogy az alhálózat az Azure virtuális gépeken futó vendég operációs rendszerben a tűzfal megfelelően van konfigurálva a helyi IP-címtartományok engedélyezett forgalmat engedélyezi.**

- **Győződjön meg arról, hogy a forgalom mennyiségének nincs közel az Azure VPN gateway számára elérhető sávszélesség korlátja.**

    Hogyan ellenőrizheti, ennek a VPN-készülék helyileg futó függ. Például RRAS használatakor a Windows Server 2012 segítségével Teljesítményfigyelő nyomon követheti a fogadott és a VPN-kapcsolaton keresztül továbbított adatok mennyiségét. Használja a *RAS teljes* objektum, jelölje be a *bájt/s* és *továbbítása-bájtok/s* számlálókat:

    ![[3]][3]

    Az eredmények (a Basic és Standard termékváltozat 100 MB/s és a magas teljesítmény termékváltozat 200 MB/s) a VPN-átjáró számára elérhető sávszélesség kell összehasonlítani:

    ![[4]][4]

- **Győződjön meg arról, hogy telepítette a megfelelő számát és a virtuális gépek méretét az alkalmazás terhelést.**

    Határozza meg, ha a virtuális gépeket az Azure VNet bármelyikét lassan működik. Ha igen, akkor lehetséges, hogy túlterheltté, lehetséges, hogy túl kevés a terhelés kezelésére, vagy előfordulhat, hogy a terheléselosztókhoz nincs megfelelően konfigurálva. Annak meghatározására, [rögzíteni és elemezni a diagnosztikai adatok][azure-vm-diagnostics]. Az eredményeket az Azure portál használatával ellenőrizheti, de sok külső eszközök is elérhető, amely is részletes információkat adnak a teljesítményadatokat.

- **Győződjön meg arról, hogy az alkalmazás lehetővé teszi a felhőalapú erőforrások hatékony használatát.**

    Eszköz alkalmazáskód annak meghatározásához, hogy alkalmazásokat a lehető legjobb felhasználását, az erőforrások létesített minden egyes virtuális gépen. Használhatja például a [Application Insights][application-insights].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez


**Prequisites.** Rendelkeznie kell egy meglévő helyi infrastruktúra már be van állítva a megfelelő hálózati berendezések.

A megoldás üzembe helyezéséhez, a következő lépésekkel.

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket: 
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "A helyszíni és az Azure infrastruktúra-áthidalás hibrid hálózati"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "A naplók az Azure-portálon"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Teljesítményszámlálók a VPN-hálózati forgalom figyelése"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Példa VPN hálózat teljesítménye ábra"