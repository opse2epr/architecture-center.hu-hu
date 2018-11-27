---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz VPN használatával
description: A jelen cikk azt ismerteti, hogyan lehet implementálni egy helyek közötti biztonságos hálózati architektúrát, amely a VPN használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.
author: RohitSharma-pnp
ms.date: 10/22/2018
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: a494ff952dd6c8be3b38c2ca7f6740a44b5b30e1
ms.sourcegitcommit: 19a517a2fb70768b3edb9a7c3c37197baa61d9b5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/26/2018
ms.locfileid: "52295667"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával

Ez a referenciaarchitektúra bemutatja, hogyan lehet kibővíteni a helyszíni hálózatot az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával. A forgalom a helyszíni hálózat és egy Azure Virtual Network (VNet) között egy IPsec VPN-alagúton halad át. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Az architektúra a következőkben leírt összetevőkből áll.

* **Helyszíni hálózat**. A cégen belül futó helyi magánhálózat.

* **VPN-berendezés**. A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás. A VPN-berendezés lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS). A támogatott VPN-berendezések listájáért és azok Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkért tekintse meg a kiválasztott eszközre vonatkozó utasításokat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben.

* **Virtuális hálózat (VNet)**. A felhőalapú alkalmazás és az Azure VPN-átjáró összetevői ugyanazon a [virtuális hálózaton][azure-virtual-network] találhatók.

* **Azure VPN Gateway**. A [VPN Gateway][azure-vpn-gateway] szolgáltatás lehetővé teszi, hogy VPN-berendezésen keresztül csatlakoztassa a virtuális hálózatot a helyszíni hálózathoz. További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket. A VPN-átjáró az alábbi elemeket tartalmazza:
  
  * **Virtuális hálózati átjáró**. Egy, a virtuális hálózathoz virtuális VPN-berendezést biztosító erőforrás. Ez felelős az adatforgalom helyszíni hálózatról virtuális hálózatra való irányításáért.
  * **Helyi hálózati átjáró**. A helyszíni VPN-készülék absztrakciója. A felhőalkalmazásról a helyszíni hálózatra irányuló hálózati forgalom ezen az átjárón halad át.
  * **Kapcsolat**. A kapcsolat olyan tulajdonságokkal rendelkezik, amelyek megadják a kapcsolat típusát (IPSec) és a helyszíni VPN-berendezéssel megosztott kulcsot a forgalom titkosításához.
  * **Átjáró-alhálózat**. A virtuális hálózati átjáró a saját alhálózatán található, amelynek számos, az alább található Javaslatok szakaszban leírt követelménynek meg kell felelnie.

* **Felhőalkalmazás**. Az Azure-ban üzemeltetett alkalmazás. Több réteget is foglalhat magában, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

* **Belső terheléselosztó**. A rendszer a VPN-átjáróról érkező hálózati forgalmat egy belső terheléselosztón keresztül irányítja a felhőalkalmazásba. A terheléselosztó az alkalmazás előtérbeli alhálózatán található.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="vnet-and-gateway-subnet"></a>Virtuális hálózat és átjáró-alhálózat

Hozzon létre egy Azure virtuális hálózatot egy, az összes szükséges erőforrás számára elegendő méretű címtérrel. Ügyeljen arra, hogy a virtuális hálózat címtere a növekedésnek is biztosítson helyet, ha van rá esély, hogy a jövőben további virtuális gépekre is szükség lesz. A virtuális hálózat címtere nem lehet átfedésben a helyszíni hálózattal. A fenti ábra például a 10.20.0.0/16 címteret használja a virtuális hálózathoz.

Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal. Ez az alhálózat a virtuális hálózat átjárójához szükséges. Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben. Lehetőleg a címtér közepére se helyezze ezt az alhálózatot. Bevált gyakorlat az átjáró-alhálózat címterét a virtuális hálózat címterének felső végén beállítani. Az ábrán is látható példa a 10.20.255.224/27 címteret használja.  Íme egy gyors eljárás a [CIDR] kiszámításához:

1. A virtuális hálózat címterében lévő változó biteket állítsa 1-re, egészen az átjáró-alhálózat által használt bitekig, majd a többit állítsa 0-ra.
2. Alakítsa át az eredményül kapott biteket decimálisra, és fejezze ki címtérként, az előtag hosszúságát pedig állítsa az átjáró-alhálózat hosszúságára.

Egy 10.20.0.0/16 IP-címtartományú virtuális hálózat esetében például az 1. lépés alkalmazása a következő eredményt adja: 10.20.0b11111111.0b11100000.  Ha ezt átváltja decimálisra, és címtérként fejezi ki, a következőt kapja: 10.20.255.224/27. 

> [!WARNING]
> Ne telepítsen virtuális gépet az átjáró-alhálózatba. Arra is ügyeljen, hogy ne rendeljen NSG-t ehhez az alhálózathoz, különben az átjáró nem fog működni.
> 
> 

### <a name="virtual-network-gateway"></a>Virtuális hálózati átjáró

Foglaljon le egy nyilvános IP-címet a virtuális hálózati átjáróhoz.

Hozza létre a virtuális hálózati átjárót az átjáró-alhálózatban, és rendelje hozzá az újonnan lefoglalt nyilvános IP-címet. Használja az igényeinek leginkább megfelelő, és a VPN-berendezés által engedélyezett átjárótípust:

- [Házirendalapú átjárót][policy-based-routing] hozzon létre, ha házirendfeltételek, például címelőtagok alapján szoros felügyeletet szeretne gyakorolni a kérelmek útválasztása fölött. A házirendalapú átjárók statikus útválasztást használnak, és csak a helyek közötti kapcsolatoknál használhatók.

- [Útvonalalapú átjárót][route-based-routing] hozzon létre, ha RRAS használatával csatlakozik a helyszíni hálózathoz, támogatja a többhelyes vagy régiók közötti kapcsolatokat, vagy virtuális hálózatok közötti kapcsolatokat implementál (beleértve a több virtuális hálózaton áthaladó útvonalakat). Az útválasztó-alapú átjárók dinamikus útválasztást használnak a hálózatok között. Ezek jobban tűrik a hálózati útvonal hibáit, mint a statikus útvonalak, mert próbálkozhatnak más útvonalakkal. Az útválasztó-alapú átjárók is csökkenthetik a felügyeleti terheket, mert az útvonalakat nem feltétlenül kell manuálisan frissíteni, ha változik a hálózati cím.

A támogatott VPN-berendezések listáját az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliances] cikkben találja meg.

> [!NOTE]
> Az átjáró létrehozása után az átjárótípusok között csak az átjáró törlésével és újbóli létrehozásával válthat.
> 
> 

Válassza ki az Azure VPN-átjáró azon termékváltozatát, amely leginkább megfelel a teljesítménybeli követelményeknek. További informayion, lásd: [átjáró-termékváltozatok][azure-gateway-skus]

> [!NOTE]
> Az alapszintű SKU nem kompatibilis az Azure ExpressRoute-tal. A [termékváltozat][changing-SKUs] az átjáró létrehozása után is módosítható.
> 
> 

A díjat az átjáró üzemképes állapotának és rendelkezésre állásának időtartama számoljuk fel. Információk: [A VPN Gateway díjszabása][azure-gateway-charges].

Ahelyett, hogy közvetlenül engedné áthaladni a kérelmeket az alkalmazás virtuális gépeire, hozzon létre útválasztási szabályokat a bejövő alkalmazásforgalmat az átjáróról a belső terheléselosztóra irányító átjáró-alhálózathoz.

### <a name="on-premises-network-connection"></a>Helyszíni hálózati kapcsolat

Hozzon létre egy helyi hálózati átjárót. Adja meg a helyszíni VPN-berendezés nyilvános IP-címét és a helyszíni hálózat címterét. Vegye figyelembe, hogy a helyszíni VPN-berendezésnek nyilvános IP-címmel kell rendelkeznie ahhoz, hogy az Azure VPN Gateway helyi hálózati átjárója hozzáférhessen. A VPN-eszköz nem helyezhető egy hálózati címfordítás (NAT) mögé.

Hozzon létre egy helyek közötti kapcsolatot a virtuális hálózati átjáróhoz és a helyi hálózati átjáróhoz. Válassza ki a helyek közötti (IPSec) kapcsolattípust, és adja meg a megosztott kulcsot. Az Azure VPN-átjáróval végzett helyek közötti titkosítás az IPSec protokollon alapul, és előre megosztott kulcsokat használ a hitelesítéshez. A kulcsot az Azure VPN-átjáró létrehozásakor adja meg. A helyszínen futó VPN-berendezést ugyanazzal a kulccsal kell konfigurálni. Más hitelesítési mechanizmusok jelenleg nem támogatottak.

Győződjön meg arról, hogy a helyszíni útválasztási infrastruktúra úgy van beállítva, hogy az Azure virtuális hálózatban lévő címekre szánt kérelmeket továbbítsa a VPN-eszközre.

Nyissa meg a felhőalkalmazás által igényelt portokat a helyszíni hálózatban.

A kapcsolatot tesztelve ellenőrizze a következőket:

* A helyszíni VPN-berendezés megfelelően irányítja-e a forgalmat a felhőalkalmazásba az Azure VPN-átjárón keresztül.
* A virtuális hálózat megfelelően visszairányítja-e a forgalmat a helyszíni hálózatba.
* A tiltott forgalom mindkét irányba megfelelően blokkolva van-e.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az alapszintű vagy standard VPN Gateway termékváltozatoktól a nagy teljesítményű VPN termékváltozatra áttérve korlátozott függőleges méretezhetőséget érhet el.

A várhatóan nagy mennyiségű VPN-forgalmat bonyolító virtuális hálózatok esetében érdemes lehet elosztani az eltérő számítási feladatokat külön kisebb virtuális hálózatokra, és mindegyikhez konfigurálni egy VPN-átjárót.

A virtuális hálózatot particionálhatja vízszintesen vagy függőlegesen is. A vízszintes particionáláshoz helyezzen át virtuálisgép-példányokat az egyes szintekről az új virtuális hálózat alhálózataiba. Ennek eredményképp minden virtuális hálózat struktúrája és működése azonos lesz. A függőleges particionáláshoz tervezze át az egyes szinteket úgy, hogy a funkcionalitás különböző logikai területekre oszoljon (például megrendelések kezelése, számlázás, ügyfélfiókok kezelése stb.). Ezután minden funkcionális terület elhelyezhető a saját virtuális hálózatában.

Ha replikál egy helyszíni Active Directory-tartományvezérlőt a virtuális hálózatban, és implementál egy DNS-t a virtuális hálózatban, azzal csökkentheti a helyszín és a felhő közötti, biztonsággal kapcsolatos és adminisztratív forgalmat. További információk: [Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][adds-extend-domain].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ha biztosítania kell, hogy az Azure VPN-átjáró folyamatosan el tudja érni a helyszíni hálózatot, implementáljon egy feladatátvételi fürtöt a helyszíni VPN-átjáróhoz.

Ha a vállalat több helyszíni hellyel rendelkezik, hozzon létre [többhelyes kapcsolatokat][vpn-gateway-multi-site] egy vagy több Azure virtuális hálózathoz. Ehhez a megközelítéshez dinamikus (útvonalalapú) útválasztásra van szükség, ezért ügyeljen arra, hogy a helyszíni VPN-átjáró támogassa ezt a szolgáltatást.

További információk a szolgáltatói szerződésekről: [A VPN Gateway szolgáltatói szerződése][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Monitorozza a helyszíni VPN-berendezésekről származó diagnosztikai információkat. Ez a folyamat a VPN-berendezés által megadott szolgáltatásoktól függ. Ha például a Windows Server 2012 útválasztási és távelérési szolgáltatását használja, akkor az [RRAS naplózástól][rras-logging].

A kapcsolati problémákra vonatkozó információk rögzítéséhez használja az [Azure VPN-átjáró-diagnosztikát][gateway-diagnostic-logs]. Ezen naplók segítségével olyan információk követhetők nyomon például, mint a kapcsolati kérelmek forrása és célja, a használt protokoll típusa és a kapcsolat létrehozásának módja (vagy a meghiúsulás oka).

Monitorozza az Azure VPN-átjáró működési naplóit az Azure Portalon elérhető auditnaplók segítségével. A helyi hálózati átjáróhoz, az Azure hálózati átjáróhoz és a kapcsolathoz külön naplók érhetők el. Ezzel az információval nyomon követhetők az átjáró módosításai, és hasznosak lehetnek, ha egy korábban funkcionáló átjáró működése valamiért leáll.

![[2]][2]

Monitorozza a kapcsolatokat, és kövesse nyomon a hibaeseményeket. Az információ rögzítéséhez és jelentéséhez használhat olyan monitorozási csomagokat, mint amilyen például a [Nagios][nagios].

## <a name="security-considerations"></a>Biztonsági szempontok

Minden VPN-átjáróhoz hozzon létre különböző megosztott kulcsot. Egy erős megosztott kulccsal könnyebb ellenállni a találgatásos támadásoknak.

> [!NOTE]
> Az Azure Key Vault jelenleg nem használható az Azure VPN-átjáró kulcsainak előzetes megosztására.
> 
> 

Győződjön meg arról, hogy a helyszíni VPN-készülék [az Azure VPN-átjáróval kompatibilis][vpn-appliance-ipsec] titkosítási módszert alkalmaz. A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja. Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják.

Ha a helyszíni VPN-berendezés olyan szegélyhálózaton (DMZ) található, amely tűzfalat tart fenn a szegélyhálózat és az internet között, akkor előfordulhat, hogy [további tűzfalszabályokat][additional-firewall-rules] kell konfigurálnia a helyek közötti VPN-kapcsolat engedélyezéséhez.

Ha a virtuális hálózaton lévő alkalmazás adatokat küld az internetre, érdemes lehet [kényszerített bújtatást implementálni][forced-tunneling] és az internetre irányuló összes forgalmat a helyszíni hálózatra irányítani. Ez a megközelítés lehetővé teszi az alkalmazás által a helyszíni infrastruktúráról kimenő kérelmek naplózását.

> [!NOTE]
> A kényszerített bújtatás hatással lehet az Azure-szolgáltatásokhoz (például a Storage Service-hez) és a Windows licenckezelőhöz való kapcsolódásra.
> 
> 


## <a name="troubleshooting"></a>Hibaelhárítás 

Általános információk a VPN-nel kapcsolatos gyakori hibák elhárításáról: [VPN-nel kapcsolatos gyakori hibák elhárítása][troubleshooting-vpn-errors].

Az alábbi javaslatok segítenek meghatározni, hogy a helyszíni VPN-berendezés megfelelően működik-e.

- **Ellenőrizze a VPN-berendezés által létrehozott naplófájlokat hibák és meghibásodások vonatkozásában.**

    Ez segít meghatározni, hogy a VPN-berendezés helyesen működik-e. Ennek az információnak a helye a berendezéstől függően változik. Ha például a Windows Server 2012 RRAS szolgáltatását használja, az alábbi PowerShell-paranccsal jelenítheti meg az RRAS szolgáltatással kapcsolatos hibaeseményekre vonatkozó információkat:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    Az egyes bejegyzések *Üzenet* tulajdonsága tartalmaz hibaleírást. Néhány gyakori példa:

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

    Az alábbi PowerShell-paranccsal lekérheti a RRAS szolgáltatáson keresztül megkísérelt csatlakozásra vonatkozó eseménynapló-adatokat is: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    Ha nem sikerül a csatlakozás, ez a napló az alábbihoz hasonló hibákat fog tartalmazni:

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

- **Ellenőrizze a kapcsolatot és az útválasztást a VPN-átjárón.**

    Előfordulhat, hogy a VPN-berendezés nem megfelelően irányítja a forgalmat az Azure VPN Gatewayen keresztül. Egy [PsPing][psping] eszközhöz hasonló eszközzel ellenőrizheti a kapcsolatot és az útválasztást a VPN-átjárón. Ha például tesztelni szeretné a csatlakozást egy helyszíni gép és egy, a virtuális hálózaton található webkiszolgáló között, futtassa az alábbi parancsot (a `<<web-server-address>>` helyére írja a webkiszolgáló címét):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Ha a helyszíni gép át tudja irányítani a forgalmat a webkiszolgálóra, az alábbihoz hasonló kimenetnek kell megjelennie:

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

    Ha a helyi számítógép nem tud kommunikálni a megadott céllal, az alábbihoz hasonló üzenetek jelennek meg:

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

- **Ellenőrizze, hogy a helyszíni tűzfal engedélyezi-e a VPN-forgalom áthaladását, és hogy a megfelelő portok vannak-e megnyitva.**

- **Ellenőrizze, hogy a helyszíni VPN-berendezés az [Azure VPN-átjáróval][vpn-appliance] kompatibilis titkosítási módszert használja-e.** A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja. Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják.

Az alábbi javaslatok segítenek megállapítani, van-e probléma az Azure VPN-átjáróval:

- **Nézze át az [Azure VPN-átjáró diagnosztikai naplóit][gateway-diagnostic-logs], hogy nincs-e nyoma problémának.**

- **Ellenőrizze, hogy az Azure VPN-átjáró és a helyszíni VPN-berendezés ugyanazt a megosztott hitelesítési kulcsot használja-e.**

    Az Azure VPN-átjáró által tárolt megosztott kulcsot az alábbi Azure CLI-paranccsal tekintheti meg:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Használja a helyszíni VPN-berendezéshez megfelelő parancsot a berendezéshez konfigurált megosztott kulcs megjelenítéséhez.

    Győződjön meg arról, hogy az Azure VPN-átjárónak helyet adó *GatewaySubnet* alhálózat nincs NSG-hez társítva.

    Az alhálózati adatokat a következő Azure CLI-paranccsal tekintheti meg:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Ügyeljen arra, hogy ne legyen *Hálózati biztonsági csoport azonosítója* nevű adatmező. Az alábbi példa bemutatja egy olyan *GatewaySubnet*-példány eredményeit, amelyhez hozzá van rendelve egy NSG (*VPN-Gateway-Group*). Emiatt előfordulhat, hogy az átjáró hibásan működik, ha vannak szabályok meghatározva az NSG-hez.

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

- **Ellenőrizze, hogy az Azure virtuális hálózat virtuális gépeinek konfigurációja engedélyezi-e a virtuális hálózaton kívülről érkező forgalmat.**

    Ellenőrizze az összes, a virtuális gépeket tartalmazó alhálózatokhoz társított NSG-szabályt. Az alábbi Azure CLI-paranccsal megtekintheti az összes NSG-szabályt:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Ellenőrizze, hogy csatlakoztatva van-e az Azure VPN-átjáró.**

    A következő Azure PowerShell-paranccsal ellenőrizheti az Azure VPN-kapcsolat aktuális állapotát. A `<<connection-name>>` paraméter a virtuális hálózati átjárót és a helyi átjárót társító Azure VPN-kapcsolat neve.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    Az alábbi kódrészletek kiemelik azt a kimenetet, amely az átjáró csatlakoztatásakor (első példa) és leválasztásakor (második példa) jön létre:

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

Az alábbi javaslatok segítenek megállapítani, hogy van-e probléma a virtuális gazdagép konfigurációjával, a hálózati sávszélesség felhasználásával vagy az alkalmazás teljesítményével:

- **Ellenőrizze, hogy az Azure-beli virtuális gépeken futó vendég operációs rendszer tűzfala helyesen van-e konfigurálva, tehát engedi-e a helyszíni IP-tartományokról kiinduló forgalmat.**

- **Ellenőrizze, hogy a forgalom mennyisége nem közelít-e az Azure VPN-átjárón rendelkezésre álló sávszélesség korlátjához.**

    Ennek az ellenőrzési módja a helyszínen futó VPN-berendezéstől függ. Ha például a Windows Server 2012 RRAS szolgáltatását használja, a teljesítményfigyelő segítségével nyomon követheti a VPN-kapcsolaton keresztül fogadott és küldött adatok mennyiségét. A *Távelérés (RAS) áttekintése* objektum segítségével válassza ki a *Fogadott bájtok/mp* és a *Küldési sebesség (bájt/s)* számlálókat:

    ![[3]][3]

    Meg kell az eredményeket hasonlítsa össze a rendelkezésre álló sávszélesség VPN-átjáróhoz (a 100 MB/s VpnGw3 termékváltozat 1,25 GB/s, az alapszintű Termékváltozat esetén):

    ![[4]][4]

- **Győződjön meg arról, hogy az alkalmazásterheléshez megfelelő számban és méretben telepített virtuális gépeket.**

    Állapítsa meg, hogy nem lassúak-e az Azure virtuális hálózat virtuális gépei. Ha igen, akkor lehet, hogy túl vannak terhelve. Elképzelhető, hogy túl kevés gép van a terhelés kezeléséhez, vagy nincsenek megfelelően konfigurálva a terheléselosztók. Ennek meghatározásához, [rögzítse és elemezze a diagnosztikai adatokat][azure-vm-diagnostics]. Az eredményeket megvizsgálhatja az Azure Portal segítségével, de számos külső féltől származó, a teljesítményadatok részleteibe bepillantást engedő eszköz is a rendelkezésére áll.

- **Ellenőrizze, hogy az alkalmazás hatékonyan használja-e a felhőerőforrásokat.**

    Az egyes virtuális gépeken futó kialakítási alkalmazáskód, amelynek feladata meghatározni, hogy az alkalmazások a lehető legjobban kihasználják-e az erőforrásokat. Használhatja például az [Application Insights][application-insights] eszközt.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése


**Előfeltételek.** Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.

A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket: 
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
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
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: https://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Helyszíni és Azure infrastruktúrákat áthidaló hibrid hálózat"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Auditnaplók az Azure Portalon"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Teljesítményszámlálók a VPN-hálózati forgalom monitorozásához"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Példa VPN-hálózat teljesítménye ábra"
