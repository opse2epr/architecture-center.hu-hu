---
title: Biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban
description: Tudnivalók biztonságos hibrid hálózati architektúra megvalósításáról az Azure-ban.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 81dea2e4439d5a01ebb88ab86dc0a59609bb7bc3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>DMZ az Azure és a helyszíni adatközpont között

Ez a referenciaarchitektúra egy biztonságos hibrid hálózatot mutat be, amely kiterjeszti a helyszíni hálózatot az Azure-ba. Az architektúra egy *szegélyhálózatot* (DMZ-t) implementál a helyszíni hálózat és egy Azure Virtual Network (VNet) között. A DMZ hálózati virtuális berendezéseket (network virtual appliance, NVA) tartalmaz, amelyek különböző biztonsági funkciókat implementálnak, például tűzfalakat és csomagvizsgálatot. A VNet kimenő forgalma kényszerített bújtatással jut el az internetre a helyszíni hálózaton keresztül, így ellenőrizhető marad.

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az architektúrának csatlakoznia kell a helyszíni adatközponthoz egy [VPN-átjáróval][ra-vpn] vagy egy [ExpressRoute][ra-expressroute]-kapcsolattal. Az architektúra gyakori használati módjai többek között a következők:

* Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.
* Olyan infrastruktúra, amelyben szükséges az Azure VNetre a helyszíni adatközpontból érkező forgalom részletes szabályozása.
* Olyan alkalmazások, amelyeknek ellenőrizniük kell a kimenő forgalmat. Ez gyakran jogszabályi követelmény számos kereskedelmi rendszer esetében, és a segítségével elkerülhető a magánjellegű információk nyilvánosságra hozatala.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

* **Helyszíni hálózat**. A vállalatban implementált helyi magánhálózat.
* **Azure Virtual Network (VNet)**. A VNet tárolja az Azure-ban futó alkalmazást és egyéb erőforrásokat.
* **Átjáró**. Az átjáró biztosítja a helyszíni hálózat útválasztói és a VNet közötti kapcsolatot.
* **Hálózati virtuális berendezés (NVA)**. A hálózati virtuális berendezés (NVA) általános kifejezés egy virtuális gépet jelent, amely olyan feladatokat lát el, mint a hozzáférés engedélyezése vagy elutasítása tűzfalként, a nagykiterjedésű hálózati (WAN-) műveletek optimalizálása (például hálózati tömörítés), egyéni útválasztás vagy egyéb hálózati funkciók.
* **Webes rétegbeli, üzleti rétegbeli és adatrétegbeli alhálózatok**. A felhőben futó, 3 rétegbeli példaalkalmazást implementáló virtuális gépeket és szolgáltatásokat futtató alhálózatok. További információk: [Windows rendszerű virtuális gépek futtatása N szintű architektúrához az Azure-ban][ra-n-tier].
* **Felhasználó által megadott útvonalak (UDR)**. A [felhasználó által megadott útvonalak][udr-overview] határozzák meg az IP-forgalom irányát az Azure VNetekben.

    > [!NOTE]
    > A VPN-kapcsolat követelményeitől függően UDR-ek használata helyett konfigurálhat Border Gateway Protocol- (BGP-) útvonalakat a forgalmat a helyszíni hálózaton keresztül visszairányító továbbítási szabályok megvalósításához.
    > 
    > 

* **Felügyeleti alhálózat**. Ez az alhálózat olyan virtuális gépeket tartalmaz, amelyek felügyeleti és monitorozási képességeket valósítanak meg a VNeten futó összetevők számára.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="access-control-recommendations"></a>A hozzáférés-vezérléssel kapcsolatos javaslatok

Alkalmazása erőforrásainak kezeléséhez alkalmazzon [szerepköralapú hozzáférés-vezérlést ][rbac] (RBAC-t). Fontolja meg a következő [egyéni szabályok][rbac-custom-roles] létrehozását:

- Egy fejlesztési és üzemeltetési szerepkör, amely rendelkezik az alkalmazás infrastruktúrájának felügyeletéhez, az alkalmazás összetevőinek üzembe helyezéséhez, valamint a virtuális gépek monitorozásához és újraindításához szükséges engedélyekkel.  

- Egy központi informatikai rendszergazdai szerepkör a hálózati erőforrások kezeléséhez és monitorozásához.

- Egy biztonsági informatikai rendszergazdai szerepkör a biztonságos hálózati erőforrások (pl. NVA-k) kezeléséhez. 

A fejlesztési és üzemeltetési és informatikai rendszergazdai szerepkörök ne rendelkezzenek hozzáféréssel az NVA-erőforrásokhoz. Az ezekhez való hozzáférést a biztonsági informatikai rendszergazdai szerepkörre kell korlátozni.

### <a name="resource-group-recommendations"></a>Erőforráscsoportra vonatkozó javaslatok

Az Azure-erőforrások (pl. virtuális gépek, virtuális hálózatok és terheléselosztók) könnyedén felügyelhetők, ha erőforráscsoportokba helyezi őket. Rendeljen RBAC-szerepköröket az egyes erőforráscsoportokhoz a hozzáférés korlátozásához.

A következő erőforráscsoportok létrehozását javasoljuk:

* Egy erőforráscsoport, amely a VNetet (a virtuális gépeken kívül), az NSG-ket és a helyszíni hálózathoz való kapcsolódáshoz szükséges átjáróerőforrásokat tartalmazza. Rendelje hozzá a központi informatikai rendszergazdai szerepkört ehhez az erőforráscsoporthoz.
* Egy erőforráscsoport, amely az NVA-k virtuális gépeit (beleértve a terheléselosztót), a jumpboxot és egyéb felügyeleti virtuális gépeket tartalmaz, valamint a forgalmat az NVA-kon keresztül kényszerítő átjáró alhálózatához tartozó UDR-t. Rendelje hozzá a biztonsági informatikai rendszergazdai szerepkört ehhez az erőforráscsoporthoz.
* Különítse el az erőforráscsoportokat a terheléselosztót és a virtuális gépeket tartalmazó alkalmazásrétegek esetében. Ügyeljen arra, hogy ez az erőforráscsoport ne tartalmazza az egyes rétegek alhálózatait. Rendelje hozzá a fejlesztési és üzemeltetési szerepkört ehhez az erőforráscsoporthoz.

### <a name="virtual-network-gateway-recommendations"></a>Virtuális hálózati átjáróra vonatkozó javaslatok

A VNet felé irányuló helyszíni forgalom egy virtuális hálózati átjárón halad keresztül. Javasoljuk egy [Azure VPN Gateway][guidance-vpn-gateway] vagy egy [Azure ExpressRoute-átjáró][guidance-expressroute] használatát.

### <a name="nva-recommendations"></a>Hálózati virtuális készülékre vonatkozó javaslatok

Az NVA-k különböző szolgáltatásokat biztosítanak a hálózati forgalom felügyeletéhez és monitorozásához. Az [Azure Marketplace][azure-marketplace-nva] számos külső féltől származó NVA-t kínál, amelyek használhatóak. Ha ezek közül egyik sem felel meg az igényeinek, létrehozhat egyéni NVA-kat a virtuális gépek használatával. 

Ezen referenciaarchitektúra megoldásának üzembe helyezése például a következő funkciókkal rendelkező NVA-t valósít meg egy virtuális gépen:

* A forgalom átirányítása [IP-továbbítással][ip-forwarding] történik az NVA hálózati adapterein.
* A forgalom csak akkor haladhat át az NVA-n, ha az megfelelő. A referenciaarchitektúrában található minden NVA virtuális gép egy egyszerű Linux-útválasztó. A bejövő forgalom az *eth0* hálózati adapteren érkezik, az egyéni szkriptek által definiált szabályoknak megfelelő kimenő forgalom pedig az *eth1* hálózati adapteren keresztül távozik.
* Az NVA-k csak a felügyeleti alhálózatról konfigurálhatók. 
* A felügyeleti alhálózatra irányított forgalom nem halad át az NVA-kon. Ellenkező esetben az NVA-k meghibásodása esetén nem lenne elérhető útvonal a felügyeleti alhálózat felé azok javításához.  
* Az NVA virtuális gépei egy [rendelkezésre állási csoportban][availability-set] vannak elhelyezve egy terheléselosztó mögött. Az átjáróban található UDR átirányítja az NVA kéréseit a terheléselosztóhoz.

Alkalmazzon egy 7. rétegbeli NVA-t, amely megszakítja az alkalmazások kapcsolatát az NVA szintjén, és fenntartja az affinitást a háttérszintekkel. Ez garantálja a szimmetrikus kapcsolatokat, ahol a háttérbeli rétegekből érkező válaszforgalom az NVA-n halad át.

Egy másik megfontolandó lehetőség több NVA soros összekapcsolása, ahol minden NVA egy speciális biztonsági feladatot hajt végre. Ez lehetővé teszi az egyes biztonsági funkciók NVA-nkénti felügyeletét. Egy tűzfalat implementáló NVA például sorba kapcsolható egy identitásszolgáltatásokat futtató NVA-val. A könnyű felügyelet ára azonban az, hogy a több hálózati ugrás növelheti a késést, ezért győződjön meg arról, hogy ez nem befolyásolja az alkalmazás teljesítményét.


### <a name="nsg-recommendations"></a>NSG-kre vonatkozó javaslatok

A VPN-átjáró közzétesz egy nyilvános IP-címet a helyszíni hálózattal való kapcsolat számára. Javasoljuk, hogy hozzon létre egy hálózati biztonsági csoportot (NSG-t) a bejövő NVA-alhálózat számára, amelynek szabályai blokkolnak minden forgalmat, amely nem a helyszíni hálózatról érkezik.

Továbbá javasoljuk, hogy minden egyes alhálózathoz használjon hálózati biztonsági csoportokat. Ezek egy további védelmi szintet biztosítanak a hibásan konfigurált vagy letiltott NVA-kat megkerülő bejövő forgalommal szemben. A referenciaarchitektúrában szereplő webes rétegbeli alhálózat például egy olyan NSG-t implementál, amely egyik szabálya figyelmen kívül hagy minden olyan forgalmat, amely nem a helyszíni hálózatról (192.168.0.0/16) vagy a VNetről érkezik, egy másik szabálya pedig figyelmen kívül hagy minden kérést, amely nem a 80-as porton érkezik.

### <a name="internet-access-recommendations"></a>Internet-hozzáférésre vonatkozó javaslatok

Használjon [kényszerített bújtatást][azure-forced-tunneling] a helyszíni hálózaton keresztüli összes kimenő internetes forgalomra egy helyek közötti VPN-alagút használatával, és irányítsa azt az internet felé hálózati címfordítás (NAT) használatával. Ez megakadályozza az adatrétegben tárolt bizalmas adatok véletlen kiszivárgását, és lehetővé teszi a teljes kimenő forgalom vizsgálatát és ellenőrzését.

> [!NOTE]
> Ne blokkolja teljesen az alkalmazásrétegből érkező internetes forgalmat, mivel az megakadályozná, hogy ezek a rétegek használhassák a nyilvános IP-címeket használó Azure PaaS szolgáltatásokat (pl. virtuális gépek diagnosztikájának naplózása, virtuálisgép-bővítmények letöltése és egyéb funkciók). Az Azure Diagnostics azt is megköveteli, hogy az összetevők olvashassanak és írhassanak az Azure Storage-fiókba.
> 
> 

Ellenőrizze, hogy a kimenő internetes forgalom kényszerített bújtatása megfelelő-e. Ha VPN-kapcsolattal csatlakozik egy helyszíni kiszolgálón található [útválasztási és távolelérési szolgáltatással][routing-and-remote-access-service], használja a [WireSharkot][wireshark], a [Microsoft Message Analyzert](https://www.microsoft.com/download/details.aspx?id=44226) vagy hasonló eszközt.

### <a name="management-subnet-recommendations"></a>Felügyeleti alhálózatokra vonatkozó javaslatok

A felügyeleti alhálózaton található jumpbox hajtja végre a felügyeleti és monitorozási funkciókat. Korlátozza az összes biztonságos felügyeleti feladat végrehajtását a jumpboxra.
 
Ne hozzon létre nyilvános IP-címet a jumpbox számára. Ehelyett hozzon létre egy útvonalat a jumpbox eléréséhez a bejövő átjárón keresztül. Hozzon létre NSG-szabályokat, hogy a felügyeleti alhálózat csak az engedélyezett útvonalról érkező kérésekre válaszoljon.

## <a name="scalability-considerations"></a>Méretezési szempontok

A referenciaarchitektúra egy terheléselosztó használatával irányítja át a helyszíni hálózati forgalmat egy NVA-eszközökből álló készletbe, amely átirányítja a forgalmat. Az NVA-k egy [rendelkezésre állási csoportban][availability-set] vannak elhelyezve. Ez a kialakítás lehetővé teszi az NVA-k teljesítményének monitorozását az időben, valamint NVA-eszközök hozzáadását a terhelés növekedése esetén.

A VPN-átjáró szabványos termékváltozata legfeljebb 100 Mb/s folyamatos teljesítményt támogat. A nagy teljesítményű termékváltozat akár 200 Mb/s teljesítményt biztosít. Magasabb sávszélesség esetén fontolja meg az ExpressRoute-átjáróra való bővítést. Az ExpressRoute akár 10 Gb/s sávszélességet és a VPN-kapcsolatnál kisebb késést biztosít.

Az Azure-átjárók skálázhatóságával kapcsolatos további információkat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn-gateway-scalability] és a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute-scalability] szóló cikk skálázási szempontokkal kapcsolatos szakaszában olvashat.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ahogy arról a korábbiakban már volt szó, a referenciaarchitektúra egy terheléselosztó mögötti NVA-eszközkészletet alkalmaz. A terheléselosztó egy állapotminta használatával monitorozza az egyes NVA-kat, és minden NVA-t eltávolít a készletből, amely nem válaszol.

Ha az Azure ExpressRoute segítségével biztosítja a kapcsolatot a VNet és a helyszíni hálózat között, [konfiguráljon egy VPN-átjárót, hogy feladatátvételt biztosítson][ra-vpn-failover] arra az esetre, ha az ExpressRoute-kapcsolat elérhetetlenné válna.

A VPN és ExpressRoute-kapcsolatok rendelkezésre állásásának fenntartásával kapcsolatos további információkat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn-gateway-availability] és a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute-availability] szóló cikk rendelkezésre állási szempontokkal kapcsolatos részében olvashat. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az alkalmazások és erőforrások monitorozása a felügyeleti alhálózaton található jumpbox feladata. Az alkalmazás követelményeitől függően előfordulhat, hogy további monitorozási erőforrásokra van szükség a felügyeleti alhálózaton. Ebben az esetben az ilyen erőforrások elérését a jumpboxon keresztül kell biztosítani.

Ha a helyszíni hálózat és az Azure közötti átjárókapcsolat nem érhető el, a jumpboxot elérheti egy nyilvános IP-cím üzembe helyezésével és a jumpboxhoz való hozzáadásával, majd az internetről való távoli eljáráshívással.

A referenciaarchitektúrában található minden réteg alhálózatát NSG-szabályok védik. Lehet, hogy létre kell hoznia egy szabályt a 3389-es port megnyitásához az RDP-hozzáféréshez (Windows rendszerű virtuális gépek esetében), vagy a 22-es port megnyitásához az SSH-hozzáféréshez (Linux rendszerű virtuális gépek esetében). Az egyéb felügyeleti és monitorozási eszközöknek szabályokra lehet szükségük további portok megnyitásához.

Ha az ExpressRoute használatával biztosítja a kapcsolatot a helyszíni adatközpont és az Azure között, az [Azure Connectivity Toolkit (AzureCT)][azurect] segítségével monitorozhatja és háríthatja el a csatlakozási hibákat.

A VPN- és ExpressRoute-kapcsolatok monitorozásával és felügyeletével kapcsolatos további információkat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn-gateway-manageability] és a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute-manageability] szóló cikkben olvashat.

## <a name="security-considerations"></a>Biztonsági szempontok

Ez a referenciaarchitektúra többszintű biztonságot valósít meg.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>Összes helyszíni felhasználói kérés átirányítása az NVA-n keresztül
Az átjáró alhálózatán található UDR blokkolja az összes olyan felhasználói kérést, amely nem a helyszínről érkezik. Az UDR átadja az engedélyezett kéréseket a privát DMZ alhálózatán lévő NVA-knak, ezek a kérések pedig továbbítva lesznek az alkalmazásnak, ha az NVA-szabályok ezt engedélyezik. További útvonalakat is hozzáadhat az UDR-hez, de győződjön meg arról, hogy azok véletlenül sem kerülik meg az NVA-kat vagy blokkolják a felügyeleti alhálózat felé irányuló felügyeleti forgalmat.

Az NVA-k előtti terheléselosztó emellett biztonsági eszközként is működik, amely figyelmen kívül hagyja a terheléselosztási szabályok szerint nem nyitott portokon keresztül érkező forgalmat. A referenciaarchitektúrában található terheléselosztók csak a 80-as porton érkező HTTP-, és a 443-as porton érkező HTTPS-kéréseket figyelik. Dokumentálja a terheléselosztóhoz hozzáadott további szabályokat, és monitorozza a forgalmat a biztonsági problémák elkerülése érdekében.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>NSG-k használata az alkalmazásrétegek közötti forgalom blokkolására/átengedésére
A rétegek közötti forgalom NSG-k használatával korlátozható. Az üzleti réteg blokkol minden forgalmat, amely nem a webes rétegből származik, az adatréteg pedig blokkol minden olyan forgalmat, amely nem az üzleti rétegből származik. Ha a követelményei miatt bővíteni kívánja az NSG-szabályokat a rétegekhez való szélesebb körű hozzáférés biztosítása érdekében, mérlegelje az ezzel járó biztonsági kockázatokat. Minden új bejövő útvonal újabb lehetőség a véletlen vagy szándékos adatvesztésre vagy az alkalmazás meghibásodására.

### <a name="devops-access"></a>Fejlesztési és üzemeltetési hozzáférés
A fejlesztési és üzemeltetési csapat által az egyes rétegekben végrehajtható műveletek az [RBAC][rbac] használatával korlátozhatók. Engedélyek megadása esetén alkalmazza a [legalacsonyabb jogosultsági szint elvét][security-principle-of-least-privilege]. Naplózzon minden felügyeleti műveletet, és rendszeresen végezzen ellenőrzést. Így meggyőződhet arról, hogy minden konfigurációmódosítás tervezett volt.

## <a name="solution-deployment"></a>A megoldás üzembe helyezése

Az ezeknek a javaslatoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a [GitHubon][github-folder]. A referenciaarchitektúra az alábbi utasításokat követve helyezhető üzembe:

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:   
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-private-dmz-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. A paraméterfájlokban szerepel egy nem módosítható rendszergazdai felhasználónév és jelszó minden virtuális gép számára. Erősen ajánlott mindkettőt azonnal lecserélni. Az üzemelő példány minden virtuális gépe esetében válassza ki a gépet az Azure Portalon, majd a **Támogatás + hibaelhárítás** panelen kattintson a **Jelszó alaphelyzetbe állítása** lehetőségre. A **Mód** legördülő listában válassza a **Jelszó alaphelyzetbe állítása** lehetőséget, majd adjon meg új értéket a **Felhasználónév** és a **Jelszó** mezőben. Kattintson a **Frissítés** gombra a mentéshez.

## <a name="next-steps"></a>További lépések

* Megtudhatja, hogyan implementálhat egy [DMZ-t az Azure és az internet között](secure-vnet-dmz.md).
* Megismerheti, hogyan implementálhat egy [magas rendelkezésre állású hibrid hálózati architektúrát][ra-vpn-failover].
* További információt a hálózati biztonság Azure-ral történő felügyeletével kapcsolatban [a Microsoft-felhőszolgáltatásokkal és a hálózati biztonsággal][cloud-services-network-security] kapcsolatos cikkben olvashat.
* Az Azure-erőforrások védelmével kapcsolatos további információkat [a Microsoft Azure biztonsági szolgáltatásait ismertető][getting-started-with-azure-security] cikkben olvashat. 
* Az Azure-átjárókapcsolatokra vonatkozó biztonsági megfontolásokkal kapcsolatos további információkat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn-gateway-security] és a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute-security] szóló cikkben olvashat.
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "Biztonságos hibrid hálózati architektúra"
