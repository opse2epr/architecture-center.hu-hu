---
title: "A biztonságos hibrid hálózati architektúra végrehajtása az Azure-ban"
description: "Hogyan egy biztonságos hibrid hálózati architektúra végrehajtásához az Azure-ban."
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 778d5ef6967a09b03bb6b5aca67e3e0c170ad016
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>DMZ Azure és a helyszíni adatközpont között

A referencia-architektúrában egy biztonságos hibrid hálózat, amely egy helyi hálózati kibővíti az Azure-bA látható. Az architektúra valósítja meg, más néven DMZ egy *szegélyhálózaton*, a helyszíni hálózat és egy Azure virtuális hálózatot (VNet) között. A Szegélyhálózaton a hálózati virtuális készülékek (NVAs), amelyek megvalósítják a biztonsági funkciók, például tűzfalak és Csomagvizsgálat tartalmazza. A vnet minden kimenő forgalom kényszerítve bújtatott a helyi hálózaton keresztül kell naplózni.

[![0]][0] 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

Ez az architektúra a helyszíni adatközpontját használatával való kapcsolatot igényel a [VPN-átjáró] [ ra-vpn] vagy egy [ExpressRoute] [ ra-expressroute] a kapcsolat. Ez az architektúra a jellemző használati többek között:

* Hibrid alkalmazások ahol munkaterhelések Futtatás részben helyszínen és részben az Azure.
* A tanúsítványhasználat pontos szabályzása egy helyszíni adatközpontot egy Azure virtuális hálózatot érkező forgalom számára szükséges infrastruktúrát.
* Alkalmazások, amelyek kell naplózni a kimenő forgalmat. Ez gyakran számos kereskedelmi rendszer szabályozó követelményt, és megakadályozhatja, hogy a személyes információk közzététele során.

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőkből áll.

* **A helyszíni hálózat**. Helyi magánhálózat szervezetként valósítva.
* **Az Azure virtuális hálózathoz (VNet)**. A virtuális hálózat üzemelteti, az alkalmazás és az egyéb Azure-beli erőforrásokhoz.
* **Átjáró**. Az átjáró a helyszíni hálózaton útválasztók és a hálózatok közötti kapcsolatot biztosít.
* **Hálózati virtuális készülék (NVA)**. NVA egy általános kifejezés, amely egy virtuális Gépet, feladatok, például a hozzáférés engedélyezésére vagy megtagadására szerint, a tűzfal, optimalizálás nagy kiterjedésű hálózati (WAN) operations (beleértve a hálózati tömörítést), egyéni, vagy más hálózati funkciókat ismerteti.
* **Webalkalmazás-rétegből, üzleti szint és adatok réteg alhálózatok**. A virtuális gépek és szolgáltatások, amelyek megvalósítják a felhőben futó 3 szintű mintaalkalmazás üzemeltető alhálózatokat. Lásd: [fut a Windows virtuális gépek az Azure-N szintű architektúra] [ ra-n-tier] további információt.
* **Felhasználó által megadott útvonalak (UDR)**. [Felhasználó által megadott útvonalak] [ udr-overview] határozza meg az Azure Vnetekhez belüli IP-forgalom áramlását.

    > [!NOTE]
    > Attól függően, hogy a VPN-kapcsolat követelményeinek konfigurálható Border Gateway Protocol (BGP) útvonalak megvalósításához a továbbítási szabályaik udr-EK használata helyett a helyi hálózaton keresztül, hogy közvetlen forgalom.
    > 
    > 

* **Felügyeleti alhálózat.** Ez az alhálózat virtuális gépek, amelyek megvalósítják a felügyeleti és megfigyelési lehetőségek a Vneten belül futó összetevőket tartalmazza.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="access-control-recommendations"></a>Hozzáférés-vezérlő javaslatok

Használjon [szerepköralapú hozzáférés-vezérlés ] [ rbac] (RBAC) az alkalmazás-erőforrások kezeléséhez. Érdemes lehet létrehozni a következő [egyéni szerepkörök][rbac-custom-roles]:

- Az alkalmazás, az infrastruktúra felügyeletéhez engedélyekkel rendelkező DevOps szerepkör az alkalmazás-összetevők telepítése és figyelése, és indítsa újra a virtuális gépek.  

- Egy központi informatikai rendszergazda szerepkör kezelni és megfigyelni a hálózati erőforrásokhoz.

- A biztonsági informatikai rendszergazda szerepkör biztonságos hálózati erőforrások, például a NVAs kezeléséhez. 

A DevOps és a informatikai rendszergazda szerepkör nem rendelkezhet az NVA erőforrásokhoz való hozzáférést. Ez az informatikai rendszergazda szerepkör biztonsági korlátozva kell lennie.

### <a name="resource-group-recommendations"></a>Erőforrás csoport javaslatok

Azure-erőforrások, például a virtuális gépek, a virtuális hálózatokat és a terheléselosztók könnyen kezelhető csoportosításával ezeket az erőforráscsoportokat. Az RBAC-szerepkörök hozzárendelése minden erőforráscsoporthoz korlátozhatja a hozzáférést.

Azt javasoljuk, hogy a következő erőforrás-csoportok létrehozása:

* A virtuális hálózat (kivéve a virtuális gépek), az NSG-k és az átjáró-erőforrásokat, a helyszíni hálózathoz való csatlakozás tartalmazó csoport. Ez az erőforráscsoport a központi IT rendszergazda szerepkör hozzárendelése.
* A NVAs (beleértve a terheléselosztó), a jumpbox és más felügyeleti virtuális gépek és az átjáró-alhálózat, amely arra kényszeríti az összes forgalmat a NVAs keresztül UDR a virtuális gépeket tartalmazó erőforráscsoport. Rendelje hozzá az erőforráscsoport informatikai rendszergazda szerepkörhöz biztonságát.
* Külön erőforráscsoportok az egyes alkalmazás rétegek, amelyek tartalmazzák a terheléselosztó és a virtuális gépek. Vegye figyelembe, hogy ez az erőforráscsoport nem tartalmazhatja az egyes rétegekhez alhálózatokat. Ez az erőforráscsoport a DevOps szerepkör hozzárendelése.

### <a name="virtual-network-gateway-recommendations"></a>Virtuális hálózati átjáró javaslatok

Helyszíni forgalom virtuális hálózati átjárón keresztül továbbítja a virtuális hálózat. Ajánlott egy [Azure VPN gateway] [ guidance-vpn-gateway] vagy egy [Azure ExpressRoute-átjáró][guidance-expressroute].

### <a name="nva-recommendations"></a>NVA javaslatok

NVAs különböző szolgáltatásokhoz a kezelése és a hálózati forgalom figyelésére. A [Azure piactér] [ azure-marketplace-nva] több külső szállító használható NVAs kínál. Ha a külső NVAs egyike megfelelnek az elvárásainak, létrehozhat egy egyéni NVA használó virtuális gépek. 

A megoldás üzembe helyezése a referenciaarchitektúra például megvalósítja NVA a virtuális gép a következő funkciókat:

* Továbbítódik használatával [IP-továbbítás] [ ip-forwarding] az NVA hálózati adapterek (NIC).
* Csak akkor, ha szükséges, ehhez az NVA továbbítása engedélyezve van a forgalom. A referencia-architektúrában NVA virtuális gépek egy egyszerű Linux útválasztó. Bejövő forgalom érkezik, a hálózati illesztő *eth0*, és a kimenő adatforgalmat a megfelelő hálózati felületen keresztül küldött egyéni parancsfájlok által megadott szabályok *eth1*.
* A NVAs csak a felügyeleti alhálózatból konfigurálható. 
* A felügyeleti alhálózathoz áthaladó forgalom nem halad át a NVAs. Ellenkező esetben ha meghibásodik a NVAs, nem lenne javítsa ki a felügyeleti alhálózatnak nincs útvonal.  
* A virtuális gépek számára az NVA kerülnek egy [rendelkezésre állási csoport] [ availability-set] egy terheléselosztó mögött. Az átjáró-alhálózat UDR NVA kérelmek terheléselosztóhoz irányítja.

A réteg-7 NVA állítsa le az alkalmazás kapcsolatok NVA szinten, majd a háttérrendszer rétegekkel affinitás karbantartása tartalmazza. Ez biztosítja, hogy szimmetrikus kapcsolat, amelyben a háttérrendszer rétegek válasz forgalmát keresztül az NVA adja vissza.

Figyelembe kell venni egy másik lehetőség az egyes NVA a speciális biztonsági feladatok sorozat, több NVAs kapcsolódik. Ez lehetővé teszi, hogy minden egyes biztonsági függvény kezelendő-NVA alapon. Például egy tűzfal végrehajtási NVA sikerült elhelyezni sorozat az identitás-szolgáltatásokat futtató NVA. Mi a fontosabb: a könnyebb kezelés érdekében, előfordulhat, hogy növelje a késés, ezért figyeljen oda arra, hogy ez nincs hatással az alkalmazás teljesítményének extra hálózati ugrásokat hozzáadását.


### <a name="nsg-recommendations"></a>NSG javaslatok

A VPN-átjáró nyilvános IP-címnek a kapcsolat a helyszíni hálózat mutatja. Javasoljuk a hálózati biztonsági csoport (NSG) létrehozása a bejövő NVA alhálózat, nem a helyi hálózatról származó összes forgalmat szabályait.

Javasoljuk továbbá szemléltetéséhez NSG-ket az egyes alhálózatokon a második szintű védelmet biztosít a bejövő forgalom megkerüli a helytelenül konfigurált vagy letiltott NVA. A webes réteg alhálózati a referencia-architektúrában például egy NSG szabállyal figyelmen kívül az összes kérelmet kapott a helyi hálózati (192.168.0.0/16) vagy a virtuális hálózat és egy másik szabály, amely figyelmen kívül hagyja az összes kérelmet nem 80-as porton nem valósítja meg.

### <a name="internet-access-recommendations"></a>Internet-hozzáférés javaslatok

[Kényszerített bújtatás] [ azure-forced-tunneling] minden kimenő internetforgalom keresztül a helyszíni hálózathoz a webhelyek VPN-alagút használatával, és az internetkapcsolat, és a hálózati címfordítás (NAT) továbbítja. Ez megakadályozza, hogy az a adatrétegbeli tárolt bizalmas adatok véletlen kiszivárgásának, és lehetővé teszi, hogy a vizsgálat és naplózás minden kimenő forgalom.

> [!NOTE]
> Nincs teljesen blokkolás alkalmazás rétegek érkező internetes forgalom, ez megakadályozza, hogy ezek a Rétegek használatával a nyilvános IP-címek, például a virtuális gép diagnosztikai naplózás, használó Azure PaaS szolgáltatások letöltésének Virtuálisgép-bővítmények és egyéb funkciókat. Az Azure diagnostics is szükséges, hogy összetevők olvashatja, és az Azure tárolási fiók írásához.
> 
> 

Győződjön meg arról, hogy kimenő internetforgalom kényszerített bújtatott megfelelően. A VPN-kapcsolat használata a [Útválasztás és távelérés szolgáltatás] [ routing-and-remote-access-service] egy helyszíni kiszolgálón eszközzel például [WireShark] [ wireshark]vagy [Microsoft Message Analyzert](https://www.microsoft.com/download/details.aspx?id=44226).

### <a name="management-subnet-recommendations"></a>Felügyeleti alhálózati javaslatok

A felügyeleti alhálózat egy kezelési és figyelési funkcióit végrehajtó jumpbox tartalmaz. Minden biztonságos felügyeleti feladatok végrehajtása a jumpbox korlátozása.
 
Ne hozzon létre egy nyilvános IP-címet a jumpbox. Ehelyett hozzon létre egy útvonalat a jumpbox eléréséhez a bejövő átjárón keresztül. NSG-szabályok létrehozása, így a felügyeleti alhálózati csak válaszol az engedélyezett útvonal.

## <a name="scalability-considerations"></a>Méretezési szempontok

A referencia-architektúrában terheléselosztót használó át tudja irányítani a helyi hálózati forgalom NVA eszközök, amelyek a forgalmat a készlethez. A NVAs kerülnek egy [rendelkezésre állási csoport][availability-set]. Ez a kialakítás teszi lehetővé az átviteli sebessége a NVAs figyelése adott idő alatt, és a terhelés növekedésével válaszul NVA eszközök hozzáadása.

A standard Termékváltozat VPN-átjáró támogatja a tartós legfeljebb 100 MB/s átviteli sebességgel. A magas teljesítmény SKU 200 MB/s biztosít. Nagyobb sávszélesség fontolja meg egy ExpressRoute-átjáró frissítése. ExpressRoute legfeljebb 10 GB/s sávszélességet biztosít a kisebb késést biztosít a VPN-kapcsolatot.

Azure-átjárók méretezhetőségét kapcsolatos további információkért lásd: a méretezhetőséget szempont szakasz [végrehajtási egy hibrid hálózati architektúra az Azure-ral és a helyszíni VPN] [ guidance-vpn-gateway-scalability] és [Egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási][guidance-expressroute-scalability].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ahogy azt korábban említettük, a referencia-architektúrában használ egy terheléselosztó mögött NVA eszközök készletét. A load balancer egy állapotmintáihoz használja minden NVA figyeléséhez, és eltávolítja a nem válaszoló NVAs a készletből.

Ha az Azure ExpressRoute segítségével létesítsen kapcsolatot a virtuális hálózat és a helyszíni hálózat között [feladatátvétel biztosítására VPN-átjáró konfigurálása] [ ra-vpn-failover] Ha ExpressRoute-kapcsolat nem érhető el. .

A VPN és a ExpressRoute rendelkezésre állás fenntartása konkrét információkért lásd: a rendelkezésre állási szempontok [végrehajtási egy hibrid hálózati architektúra az Azure-ral és a helyszíni VPN] [ guidance-vpn-gateway-availability] és [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási][guidance-expressroute-availability]. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Összes alkalmazás- és erőforrás-figyelés felügyeleti alhálózaton jumpbox kell elvégezni. Az alkalmazás követelményeitől függően szükség lehet a felügyeleti alhálózatban lévő további figyelési erőforrások. Ha igen, ezeket az erőforrásokat a jumpbox használatával kell elérni.

Ha az Azure-bA a helyi hálózati átjáró kapcsolat nem működik, továbbra is érhető el a jumpbox egy nyilvános IP-címet, üzembe helyezésével hozzáadná a jumpbox, és a távoli eljáráshívás az internetről.

A referencia-architektúrában egyes rétegek alhálózati NSG-szabályok védik. Hozzon létre egy szabályt, amely nyissa meg a távoli asztal protokoll (RDP) hozzáférésre Windows virtuális gépek a 3389-es port vagy a secure shell (SSH) hozzáférésre a Linux virtuális gépek 22-es portot szeretne. Más felügyeleti és figyelési eszközök további portok megnyitására szolgáló szabályok lehet szükség.

Ha ExpressRoute segítségével adja meg a helyszíni adatközpontját és az Azure közötti kapcsolat, használja a [Azure kapcsolat Toolkit (AzureCT)] [ azurect] és csatlakozási problémák hibaelhárítása .

Kifejezetten célzó megfigyelését és felügyeletét a cikkekben VPN- és ExpressRoute kapcsolatot további információkért [végrehajtási egy hibrid hálózati architektúra az Azure-ral és a helyszíni VPN] [ guidance-vpn-gateway-manageability] és [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási][guidance-expressroute-manageability].

## <a name="security-considerations"></a>Biztonsági szempontok

A referencia-architektúrában több szintű biztonságot valósítja meg.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Minden helyi felhasználói kérelmek keresztül az NVA Útválasztás
Az átjáró-alhálózat UDR helyszíni kapott kivételével az összes felhasználói kérelmek blokkolja. A UDR fázisok engedélyezett a NVAs kérelmek titkos DMZ az alhálózaton, és ezek a kérelmek kerülnek átadásra az alkalmazás Ha az NVA szabályok által engedélyezett számukra. Más útvonalakat hozzáadni a UDR, de győződjön meg arról, hogy azok nem véletlenül figyelmen kívül hagyása a NVAs vagy a felügyeleti alhálózati szánt felügyeleti kommunikációt blokkoló.

A terheléselosztó a NVAs előtt is funkcionál biztonsági eszköz figyelmen kívül hagyja a forgalom a portokon, amelyek nincsenek nyissa meg a terheléselosztási szabályok. A referencia-architektúrában azokat a terheléselosztókat csak figyeljen a HTTP-kérelmek 80-as porton és a 443-as portot a HTTPS-kéréseket. A dokumentum további szabályokat, hogy a terheléselosztó hozzáadása, és annak érdekében, hogy nincsenek biztonsági problémák forgalom figyelését.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>Az NSG-k segítségével forgalom blokkolása/fázis alkalmazásrétegek közötti
Rétegek közötti forgalom NSG-k használatával van korlátozva. Az üzleti szint blokkol minden forgalom a webes réteg nem származnak, és az adatszint blokkol nem érkeznek, az üzleti szinten lévő összes forgalom. Ha bontsa ki az NSG-szabályok, hogy ezek a rétegek szélesebb körű hozzáférjenek követelmény, mérjük a biztonsági kockázatok elleni követelménynek. Minden egyes új bejövő út véletlen vagy szándékos adatok kiszivárgásának vagy alkalmazás károkért lehetőséget jelenti.

### <a name="devops-access"></a>DevOps hozzáférés
Használjon [RBAC] [ rbac] DevOps egyes rétegek végezhető műveletek korlátozására. Engedélyeket ad, ha a [legalacsonyabb jogosultsági szint elve][security-principle-of-least-privilege]. Jelentkezzen az összes felügyeleti műveletet, és végezze el annak érdekében, hogy a konfigurációs változásokat tervezett rendszeres ellenőrzéseket.

## <a name="solution-deployment"></a>A megoldás üzembe helyezése

A környezet egy referencia-architektúrában, amely megvalósítja az alábbi javaslatokat érhető el a [GitHub][github-folder]. A referencia-architektúrában az alábbi utasításokat követve telepíthető:

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:   
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-private-dmz-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. A paraméter fájlok közé tartoznak kódolt rendszergazda felhasználónevét és jelszavát az összes virtuális gépet, és erősen ajánlott, hogy azonnal módosíthatja is. Az egyes virtuális gépek, a központi telepítésben lévő, válassza ki azt az Azure portálon, és kattintson a **jelszó-átállítási** a a **támogatási + hibaelhárítási** panelen. Válassza ki **jelszó-átállítási** a a **mód** legördülő mezőben, majd válasszon egy új **felhasználónév** és **jelszó**. Kattintson a **frissítés** gombra kattintva mentse.

## <a name="next-steps"></a>Következő lépések

* Megtudhatja, hogyan végrehajtásához egy [Azure és az Internet között DMZ](secure-vnet-dmz.md).
* Megtudhatja, hogyan végrehajtásához egy [magas rendelkezésre állású hibrid hálózati architektúra][ra-vpn-failover].
* Hálózati biztonság Azure-ral kezelésével kapcsolatos további információkért lásd: [Microsoft cloud services és a hálózati biztonság][cloud-services-network-security].
* Az Azure-erőforrások védelme kapcsolatos részletes információkért lásd: [Ismerkedés a Microsoft Azure biztonsági][getting-started-with-azure-security]. 
* A biztonsági aggályokon kezelése az Azure gateway-kapcsolatot további részletekért lásd: [végrehajtási egy hibrid hálózati architektúra az Azure-ral és a helyszíni VPN] [ guidance-vpn-gateway-security] és [ A hibrid hálózati architektúra az Azure ExpressRoute végrehajtási][guidance-expressroute-security].
> 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "biztonságos hibrid hálózati architektúra"
