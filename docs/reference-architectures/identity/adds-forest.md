---
title: AD DS-erőforráserdő létrehozása az Azure-ban
description: >-
  Megbízható Active Directory-tartomány létrehozása az Azure-ban.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: e32a6420821e70c84e77d2c39614f0c45efbb7e2
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Active Directory Domain Services- (AD DS-) erőforráserdő létrehozása az Azure-ban

A referenciaarchitektúra bemutatja, hogyan hozható létre egy külön Active Directory-tartomány az Azure-ban, amelyet a helyszíni AD-erdőben található tartományok megbízhatónak tekintenek. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az Active Directory Domain Services (AD DS) hierarchikus struktúrában tárolja az identitásadatokat. A hierarchikus struktúra legfelső szintű csomópontját erdőnek hívják. Az erdő tartományokat, a tartományok pedig egyéb típusú objektumokat tartalmaznak. Ez a referenciaarchitektúra egy AD DS-erdőt hoz létre az Azure-ban, amely egyirányú kimenő bizalmi kapcsolattal rendelkezik egy helyszíni tartománnyal. Az Azure-ban lévő erdő tartalmaz egy olyan tartományt, amely a helyszínen nem létezik. A bizalmi kapcsolat miatt a helyszíni tartományokra történő bejelentkezések megbízhatók a külön Azure-tartományban lévő erőforrások eléréséhez. 

Az architektúra gyakori használati módjai például a felhőben tárolt objektumok és identitások biztonsági elkülönítésének fenntartása, illetve az egyes tartományok migrálása a helyszínről a felhőbe. 

További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations]. 

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőket tartalmazza.

* **Helyszíni hálózat**. A helyszíni hálózat a saját Active Directory-erdőjét és tartományait tartalmazza.
* **Active Directory-kiszolgálók**. Ezek a felhőben virtuális gépként futó tartományvezérlők, amelyek tartományi szolgáltatásokat biztosítanak. Ezek a kiszolgálók egy egy vagy több tartományt tartalmazó erdőt üzemeltetnek a helyszíniektől elkülönítve.
* **Egyirányú bizalmi kapcsolat**. Az ábrán az Azure-ban lévő tartományból a helyszíni tartományba tartó egyirányú bizalmi kapcsolatra látható egy példa. Ez a kapcsolat lehetővé teszi a helyszíni felhasználók számára, hogy elérjék az Azure-ban lévő tartomány erőforrásait, de fordított irányban ez nem lehetséges. Kétirányú megbízhatósági kapcsolat is létrehozható, ha a felhőalapú felhasználóknak is hozzáférést kell biztosítani a helyszíni erőforrásokhoz.
* **Active Directory-alhálózat**. Az AD DS-kiszolgálók külön alhálózaton üzemelnek. A hálózati biztonsági csoport (NSG) szabályai megvédik az AD DS-kiszolgálókat, és tűzfalat biztosítanak a nem várt forrásból érkező adatforgalom ellen.
* **Azure-átjáró**. Az Azure-átjáró kapcsolatot biztosít a helyszíni hálózat és az Azure-beli virtuális hálózat között. Ez lehet [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute]. További információ: [Biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Javaslatok

Az Active Directory Azure-ban való megvalósítására vonatkozó konkrét ajánlásokért tekintse meg az alábbi cikkeket:

- [Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][adds-extend-domain]. 
- [Útmutató a Windows Server Active Directory szolgáltatás Azure-beli virtuális gépekre való telepítéséhez][ad-azure-guidelines].

### <a name="trust"></a>Bizalmi kapcsolat

A helyszíni tartományokat nem ugyanaz az erdő tartalmazza, mint a felhőben lévőket. A helyszíni felhasználók felhőbeli hitelesítésének engedélyezéséhez az Azure-ban lévő tartományoknak meg kell bízniuk a helyszíni erdőben található bejelentkezési tartományban. Hasonlóképpen, ha a felhő bejelentkezési tartományt biztosít a külső felhasználók számára, szükség lehet arra, hogy a helyszíni erdő megbízzon a felhőtartományban.

Bizalmi kapcsolatokat létesíthet erdőszinten [erdőszintű bizalmi kapcsolatok létrehozásával][creating-forest-trusts] vagy tartományszinten [külső bizalmi kapcsolatok létrehozásával][creating-external-trusts]. Az erdőszintű bizalmi kapcsolat két erdő összes tartománya között kapcsolatot hoz létre. A külső tartományszintű bizalmi kapcsolat csak két adott tartomány között hoz létre kapcsolatot. Különböző erdőkben lévő tartományok között csak külső tartományszintű bizalmi kapcsolatot hozzon létre.

A bizalmi kapcsolatok lehetnek egyirányúak vagy kétirányúak:

* Az egyirányú bizalmi kapcsolat lehetővé teszi egy tartományban vagy erdőben (*bejövő* tartomány vagy erdő) lévő felhasználóknak, hogy hozzáférjenek a másikban (*kimenő* tartomány vagy erdő) tárolt erőforrásokhoz.
* A kétirányú bizalmi kapcsolat lehetővé teszi a tartományban vagy az erdőben lévő felhasználók számára, hogy hozzáférjenek a másikban tárolt erőforrásokhoz.

A következő táblázat összefoglalja a bizalmi kapcsolati konfigurációkat néhány egyszerű forgatókönyvhöz:

| Forgatókönyv | Helyszíni bizalmi kapcsolat | Felhőbeli bizalmi kapcsolat |
| --- | --- | --- |
| A helyszíni felhasználóknak hozzá kell férniük a felhőben lévő erőforrásokhoz, de fordított irányban ez nem szükséges |Egyirányú, bejövő |Egyirányú, kimenő |
| A felhőbeli felhasználóknak hozzá kell férniük a helyszíni erőforrásokhoz, de fordított irányban ez nem szükséges |Egyirányú, kimenő |Egyirányú, bejövő |
| A felhőben lévő és a helyszíni felhasználóknak is hozzá kell férniük a felhőben és a helyszínen tárolt erőforrásokhoz is |Kétirányú, bejövő és kimenő |Kétirányú, bejövő és kimenő |

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Active Directory automatikusan skálázható az ugyanazon tartományba tartozó tartományvezérlők esetén. A kérelmek a tartományok összes vezérlője között oszlanak meg. Ha hozzáad egy másik tartományvezérlőt, ez automatikusan szinkronizálódik a tartománnyal. Ne konfiguráljon külön terheléselosztót arra, hogy a tartományban lévő vezérlőkre irányítsa a forgalmat. Győződjön meg arról, hogy minden tartományvezérlő elegendő memória-és tároló-erőforrással rendelkezik a tartomány-adatbázis kezeléséhez. Minden tartományvezérlő virtuális gépet ugyanakkora méretűre állítson be.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Építsen ki legalább két tartományvezérlőt minden tartományhoz. Ez lehetővé teszi a kiszolgálók közötti automatikus replikációt. Hozzon létre egy rendelkezésre állási csoportot az egyes tartományokat kezelő, Active Directory-kiszolgálóként működő virtuális gépekhez. Helyezzen el legalább két kiszolgálót ebben a rendelkezésre állási csoportban.

Vegye fontolóra, hogy minden tartományban kijelöljön egy vagy több kiszolgálót [készenléti műveleti főkiszolgálónak][standby-operations-masters], arra az esetre, ha nem sikerülne csatlakozni a mozgó egyedüli főkiszolgálói (FSMO) szerepkört betöltő kiszolgálóhoz.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

További információ a kezelési és monitorozási szempontokról: [Az Active Directory kiterjesztése az Azure-ra][adds-extend-domain]. 
 
További információ: [Az Active Directory monitorozása][monitoring_ad]. A feladatok megkönnyítéséhez olyan eszközöket is telepíthet a kezelési alhálózatban lévő monitorozási kiszolgálókra, mint a [Microsoft Systems Center][microsoft_systems_center].

## <a name="security-considerations"></a>Biztonsági szempontok

Az erdőszintű bizalmi kapcsolatok tranzitívak. Ha erdőszintű bizalmi kapcsolatot hoz létre egy helyszíni és egy felhőben lévő erdő között, a bizalmi kapcsolat az ezekben az erdőkben létrehozott új tartományokra is kiterjed. Ha biztonsági okokból tartományokat használ az elkülönítéshez, érdemes lehet csak tartományszintű bizalmi kapcsolatokat létrehoznia. A tartományszintű bizalmi kapcsolatok nem tranzitívak.

Active Directory-specifikus biztonsági szempontokért tekintse meg [Az Active Directory Azure-ra való kiterjesztését][adds-extend-domain] ismertető cikk biztonsági szempontokról szóló szakaszát.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ezen referenciaarchitektúra üzembe helyezéséhez rendelkezésre áll egy megoldás a [GitHubon][github]. A megoldást üzembe helyező Powershell-szkript futtatásához az Azure CLI legújabb verziójára lesz szükség. A referenciaarchitektúra üzembe helyezéséhez kövesse az alábbi lépéseket:

1. Töltse le vagy klónozza a megoldásmappát a [GitHubról][github] a helyi számítógépére.

2. Nyissa meg az Azure CLI-t, és lépjen a helyi megoldásmappára.

3. Futtassa az alábbi parancsot:
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Cserélje le a `<subscription id>` értékét a saját Azure-előfizetése azonosítójára.
   
    A `<location>` paraméter esetében adjon meg egy Azure-régiót (pl. `eastus` vagy `westus`).
   
    A `<mode>` paraméter az üzembe helyezés részletességét vezérli. Értékei a következők lehetnek:
   
   * `Onpremise`: a szimulált helyszíni környezetet helyezi üzembe.
   * `Infrastructure`: a virtuális hálózat infrastruktúráját és a jump boxot helyezi üzembe az Azure-ban.
   * `CreateVpn`: az Azure-beli virtuális hálózati átjárót telepíti, és csatlakoztatja a szimulált a helyszíni hálózathoz.
   * `AzureADDS`: üzembe helyezi az Active Directory DS-kiszolgálóként működő virtuális gépeket, telepíti az Active Directoryt ezeken a virtuális gépeken, és üzembe helyezi a tartományt az Azure-ban.
   * `WebTier`: üzembe helyezi a webes szintű virtuális gépeket és a terheléselosztót.
   * `Prepare`: az összes fenti környezetet üzembe helyezi. **Ez a lehetőség ajánlott, ha nem rendelkezik meglévő helyszíni hálózattal, de telepíteni kívánja a fentebb ismertetett teljes referenciaarchitektúrát tesztelés vagy értékelés céljából.** 
   * `Workload`: üzembe helyezi az üzleti és adatszintű virtuális gépeket és a terheléselosztókat. Vegye figyelembe, hogy ezeket a virtuális gépeket a `Prepare` üzembe helyezés nem tartalmazza.

4. Várjon, amíg az üzembe helyezés befejeződik. Ha a `Prepare` üzembe helyezést választja, a folyamat több órát vesz igénybe.
     
5. Ha szimulált helyszíni konfigurációt használ, konfigurálja a bejövő bizalmi kapcsolatot:
   
   1. Csatlakozzon a jump boxhoz (<em>ra-adtrust-mgmt-vm1</em> a <em>ra-adtrust-security-rg</em> erőforráscsoportban). Jelentkezzen be <em>testuser</em> felhasználónévvel és az <em>AweS0me@PW</em> jelszóval.
   2. A jump boxon nyisson meg egy RDP-munkamenetet a <em>contoso.com</em> tartomány (a helyszíni tartomány) első virtuális gépén. A virtuális gép IP-címe: 192.168.0.4. A felhasználónév <em>contoso\testuser</em>, a jelszó pedig <em>AweS0me@PW</em>.
   3. A *treyresearch.com* tartományból bejövő bizalmi kapcsolat létrehozásához töltse le és futtassa az [incoming-trust.ps1][incoming-trust] szkriptet.

6. Ha saját helyszíni infrastruktúrát használ:
   
   1. Töltse le az [incoming-trust.ps1][incoming-trust] szkriptet.
   2. Szerkessze a szkriptet, és cserélje le a `$TrustedDomainName` változó értékét a saját tartománya nevére.
   3. Futtassa a szkriptet.

7. A jump boxról csatlakozzon a <em>treyresearch.com</em> tartomány (a felhőbeli lévő tartomány) első virtuális gépéhez. A virtuális gép IP-címe: 10.0.4.4. A felhasználónév <em>treyresearch\testuser</em>, a jelszó pedig <em>AweS0me@PW</em>.

8. A *treyresearch.com* tartományból bejövő bizalmi kapcsolat létrehozásához töltse le és futtassa az [outgoing-trust.ps1][outgoing-trust] szkriptet. Ha a saját helyszíni gépeket használ, először szerkessze a szkriptet. A `$TrustedDomainName` változót állítsa a helyszíni tartomány nevére, és adja meg az adott tartományhoz tartozó Active Directory DS-kiszolgálók IP-címeit a `$TrustedDomainDnsIpAddresses` változóban.

9. Várjon néhány percet, amíg az előző lépések befejeződnek, majd csatlakozzon egy helyszíni virtuális géphez, és végezze el a [bizalmi kapcsolat ellenőrzését][verify-a-trust] ismertető cikk lépéseit annak megállapításához, hogy a *contoso.com* és a *treyresearch.com* tartományok közötti bizalmi kapcsolat megfelelően van-e konfigurálva.

## <a name="next-steps"></a>További lépések

* A [helyszíni AD DS-tartomány Azure-ra való kiterjesztésére][adds-extend-domain] vonatkozó ajánlott eljárások megismerése
* Az Azure-alapú [AD FS-infrastruktúra létrehozására][adfs] vonatkozó ajánlott eljárások megismerése.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Biztonságos hibrid hálózati architektúra külön Active Directory-tartományokkal"