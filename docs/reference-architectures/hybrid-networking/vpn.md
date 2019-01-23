---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz VPN használatával
titleSuffix: Azure Reference Architectures
description: Az Azure virtuális hálózat és a egy helyszíni hálózathoz egy VPN-nel csatlakoztatott kiterjedő helyek közötti biztonságos hálózati architektúra megvalósítása.
author: RohitSharma-pnp
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 515cd3f5d23957e0e153c9d25198b3cb98b92a5d
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487970"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával

Ez a referenciaarchitektúra bemutatja, hogyan lehet kibővíteni a helyszíni hálózatot az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával. A forgalom a helyszíni hálózat és egy Azure Virtual Network (VNet) között egy IPsec VPN-alagúton halad át. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![A helyszíni és Azure infrastruktúrákat áthidaló hibrid hálózat](./images/vpn.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

- **Helyszíni hálózat**. A cégen belül futó helyi magánhálózat.

- **VPN-berendezés**. A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás. A VPN-berendezés lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS). A támogatott VPN-berendezések listájáért és azok Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkért tekintse meg a kiválasztott eszközre vonatkozó utasításokat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben.

- **Virtuális hálózat (VNet)**. A felhőalapú alkalmazás és az Azure VPN-átjáró összetevői ugyanazon a [virtuális hálózaton][azure-virtual-network] találhatók.

- **Azure VPN Gateway**. A [VPN Gateway][azure-vpn-gateway] szolgáltatás lehetővé teszi, hogy VPN-berendezésen keresztül csatlakoztassa a virtuális hálózatot a helyszíni hálózathoz. További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket. A VPN-átjáró az alábbi elemeket tartalmazza:

  - **Virtuális hálózati átjáró**. Egy, a virtuális hálózathoz virtuális VPN-berendezést biztosító erőforrás. Ez felelős az adatforgalom helyszíni hálózatról virtuális hálózatra való irányításáért.
  - **Helyi hálózati átjáró**. A helyszíni VPN-készülék absztrakciója. A felhőalkalmazásról a helyszíni hálózatra irányuló hálózati forgalom ezen az átjárón halad át.
  - **Kapcsolat**. A kapcsolat olyan tulajdonságokkal rendelkezik, amelyek megadják a kapcsolat típusát (IPSec) és a helyszíni VPN-berendezéssel megosztott kulcsot a forgalom titkosításához.
  - **Átjáró-alhálózat**. A virtuális hálózati átjáró a saját alhálózatán található, amelynek számos, az alább található Javaslatok szakaszban leírt követelménynek meg kell felelnie.

- **Felhőalkalmazás**. Az Azure-ban üzemeltetett alkalmazás. Több réteget is foglalhat magában, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze. További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].

- **Belső terheléselosztó**. A rendszer a VPN-átjáróról érkező hálózati forgalmat egy belső terheléselosztón keresztül irányítja a felhőalkalmazásba. A terheléselosztó az alkalmazás előtérbeli alhálózatán található.

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

### <a name="virtual-network-gateway"></a>Virtuális hálózati átjáró

Foglaljon le egy nyilvános IP-címet a virtuális hálózati átjáróhoz.

Hozza létre a virtuális hálózati átjárót az átjáró-alhálózatban, és rendelje hozzá az újonnan lefoglalt nyilvános IP-címet. Használja az igényeinek leginkább megfelelő, és a VPN-berendezés által engedélyezett átjárótípust:

- [Házirendalapú átjárót][policy-based-routing] hozzon létre, ha házirendfeltételek, például címelőtagok alapján szoros felügyeletet szeretne gyakorolni a kérelmek útválasztása fölött. A házirendalapú átjárók statikus útválasztást használnak, és csak a helyek közötti kapcsolatoknál használhatók.

- [Útvonalalapú átjárót][route-based-routing] hozzon létre, ha RRAS használatával csatlakozik a helyszíni hálózathoz, támogatja a többhelyes vagy régiók közötti kapcsolatokat, vagy virtuális hálózatok közötti kapcsolatokat implementál (beleértve a több virtuális hálózaton áthaladó útvonalakat). Az útválasztó-alapú átjárók dinamikus útválasztást használnak a hálózatok között. Ezek jobban tűrik a hálózati útvonal hibáit, mint a statikus útvonalak, mert próbálkozhatnak más útvonalakkal. Az útválasztó-alapú átjárók is csökkenthetik a felügyeleti terheket, mert az útvonalakat nem feltétlenül kell manuálisan frissíteni, ha változik a hálózati cím.

A támogatott VPN-berendezések listáját az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliances] cikkben találja meg.

> [!NOTE]
> Az átjáró létrehozása után az átjárótípusok között csak az átjáró törlésével és újbóli létrehozásával válthat.
>

Válassza ki az Azure VPN-átjáró azon termékváltozatát, amely leginkább megfelel a teljesítménybeli követelményeknek. További információkért lásd: [átjáró-termékváltozatok][azure-gateway-skus]

> [!NOTE]
> Az alapszintű SKU nem kompatibilis az Azure ExpressRoute-tal. A [termékváltozat][changing-SKUs] az átjáró létrehozása után is módosítható.
>

A díjat az átjáró üzemképes állapotának és rendelkezésre állásának időtartama számoljuk fel. Információk: [A VPN Gateway díjszabása][azure-gateway-charges].

Ahelyett, hogy közvetlenül engedné áthaladni a kérelmeket az alkalmazás virtuális gépeire, hozzon létre útválasztási szabályokat a bejövő alkalmazásforgalmat az átjáróról a belső terheléselosztóra irányító átjáró-alhálózathoz.

### <a name="on-premises-network-connection"></a>Helyszíni hálózati kapcsolat

Hozzon létre egy helyi hálózati átjárót. Adja meg a helyszíni VPN-berendezés nyilvános IP-címét és a helyszíni hálózat címterét. Vegye figyelembe, hogy a helyszíni VPN-berendezésnek nyilvános IP-címmel kell rendelkeznie ahhoz, hogy az Azure VPN Gateway helyi hálózati átjárója hozzáférhessen. A VPN-eszköz nem helyezhető egy hálózati címfordítás (NAT) mögé.

Hozzon létre egy helyek közötti kapcsolatot a virtuális hálózati átjáróhoz és a helyi hálózati átjáróhoz. Válassza ki a helyek közötti (IPSec) kapcsolattípust, és adja meg a megosztott kulcsot. Az Azure VPN-átjáróval végzett helyek közötti titkosítás az IPSec protokollon alapul, és előre megosztott kulcsokat használ a hitelesítéshez. A kulcsot az Azure VPN-átjáró létrehozásakor adja meg. A helyszínen futó VPN-berendezést ugyanazzal a kulccsal kell konfigurálni. Más hitelesítési mechanizmusok jelenleg nem támogatottak.

Győződjön meg arról, hogy a helyszíni útválasztási infrastruktúra úgy van beállítva, hogy az Azure virtuális hálózatban lévő címekre szánt kérelmeket továbbítsa a VPN-eszközre.

Nyissa meg a felhőalkalmazás által igényelt portokat a helyszíni hálózatban.

A kapcsolatot tesztelve ellenőrizze a következőket:

- A helyszíni VPN-berendezés megfelelően irányítja-e a forgalmat a felhőalkalmazásba az Azure VPN-átjárón keresztül.
- A virtuális hálózat megfelelően visszairányítja-e a forgalmat a helyszíni hálózatba.
- A tiltott forgalom mindkét irányba megfelelően blokkolva van-e.

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

![Auditnaplók az Azure Portalon](../_images/guidance-hybrid-network-vpn/audit-logs.png)

Monitorozza a kapcsolatokat, és kövesse nyomon a hibaeseményeket. Az információ rögzítéséhez és jelentéséhez használhat olyan monitorozási csomagokat, mint amilyen például a [Nagios][nagios].

## <a name="security-considerations"></a>Biztonsági szempontok

Minden VPN-átjáróhoz hozzon létre különböző megosztott kulcsot. Egy erős megosztott kulccsal könnyebb ellenállni a találgatásos támadásoknak.

> [!NOTE]
> Az Azure Key Vault jelenleg nem használható az Azure VPN-átjáró kulcsainak előzetes megosztására.
>

Győződjön meg arról, hogy a helyszíni VPN-készülék [az Azure VPN-átjáróval kompatibilis][vpn-appliance-ipsec] titkosítási módszert alkalmaz. A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja. Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják.

Ha a helyszíni VPN-berendezés olyan szegélyhálózaton (DMZ) található, amely tűzfalat tart fenn a szegélyhálózat és az internet között, akkor előfordulhat, hogy [további tűzfalszabályokat][additional-firewall-rules] kell konfigurálnia a helyek közötti VPN-kapcsolat engedélyezéséhez.

Ha a virtuális hálózaton lévő alkalmazás adatokat küld az internetre, érdemes lehet [kényszerített bújtatást implementálni][forced-tunneling] és az internetre irányuló összes forgalmat a helyszíni hálózatra irányítani. Ez a megközelítés lehetővé teszi az alkalmazás által a helyszíni infrastruktúráról kimenő kérelmek naplózását.

> [!NOTE]
> A kényszerített bújtatás hatással lehet az Azure-szolgáltatásokhoz (például a Storage Service-hez) és a Windows licenckezelőhöz való kapcsolódásra.
>

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

**Előfeltételek**. Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.

A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

<!-- markdownlint-disable MD033 -->

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:
   - Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-rg` karakterláncot.
   - Válassza ki a régiót a **Hely** legördülő listából.
   - Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   - Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   - Kattintson a **Vásárlás** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.

<!-- markdownlint-enable MD033 -->

A kapcsolat hibaelhárítása: [egy hibrid VPN-kapcsolat hibaelhárítása](./troubleshoot-vpn.md).

<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[forced-tunneling]: /azure/vpn-gateway/vpn-gateway-about-forced-tunneling
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[azure-cli]: /cli/azure/install-azure-cli
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
