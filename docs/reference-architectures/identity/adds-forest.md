---
title: "Hozzon létre egy Active Directory tartományi szolgáltatások Erőforráserdő az Azure-ban"
description: "Hogyan hozhat létre a megbízható Active Directory-tartományban az Azure-ban.\nútmutatást, vpn-átjáró, expressroute, terheléselosztó, virtuális hálózat, active Directoryval"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: b946afa91e8bd303c51f97e18be170c4105cc8c5
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2017
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Hozzon létre egy Active Directory tartományi szolgáltatások (AD DS) Erőforráserdő az Azure-ban

A referencia-architektúrában mutatja egy külön Active Directory-tartomány létrehozása a megbízható tartományok által a helyszíni Azure Active Directory-erdőben. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Active Directory tartományi szolgáltatások (AD DS) tárolja az azonosító adatok hierarchikus. A hierarchikus struktúrában a legfelső csomópontra erdő néven ismert. Erdő-tartományt tartalmaz, és a tartományok más típusú objektumokat tartalmazzák. A referencia-architektúrában az AD DS-erdő az Azure-ban egy helyszíni tartományban kimenő egyirányú megbízhatósági kapcsolatot hoz létre. Az Azure-ban az erdő, amely nem létezik a helyi tartományt tartalmaz. A megbízhatósági kapcsolat miatt bejelentkezések a helyi tartományok ellen lehet megbízható a különböző Azure-tartományban lévő erőforrások eléréséhez. 

Ez az architektúra a gyakori felhasználási tartalmaznak objektumok és az identitások, a felhőben tárolt, és a helyszíni egyéni tartományok telepít át a felhő biztonsági elkülönítést karbantartása. 

További szempontokat lásd: [megoldás választása a integrálása a helyszíni Active Directoryról szinkronizálva az Azure][considerations]. 

## <a name="architecture"></a>Architektúra

Az architektúra a következő részből áll.

* **A helyszíni hálózat**. A helyszíni hálózat saját Active Directory-erdő és a tartományok tartalmazza.
* **Active Directory-kiszolgálók**. Ezek a tartományvezérlők végrehajtási tartományi szolgáltatások futtatásához használt virtuális gép a felhőben. Ezek a kiszolgálók üzemeltetésére egy erdőt tartalmazó egy vagy több tartományt, ezek a helyszínen elkülönül.
* **Egyirányú bizalmi kapcsolat**. Az az ábrán példában egy egyirányú bizalmi kapcsolatban a tartományból az Azure-ban a helyi tartományhoz. Ez a kapcsolat lehetővé teszi, hogy a helyszíni felhasználók az Azure-ban a tartományhoz, de nem megfordítva már erőforrásaihoz. Akkor lehet kétirányú megbízhatósági kapcsolat létrehozására, ha a felhőbeli felhasználók is hozzá kell férniük a helyszíni erőforrásokhoz.
* **Active Directory-alhálózathoz**. Az Active Directory tartományi szolgáltatások kiszolgálókon tárolt külön alhálózathoz. Hálózati biztonsági csoport (NSG) szabályok az Active Directory tartományi szolgáltatások kiszolgálók védelméhez, és adja meg a tűzfal váratlan forrásból ellen.
* **Az Azure átjáró**. Az Azure átjáró biztosítja a kapcsolatot a helyszíni hálózat és az Azure VNet között. Ez lehet egy [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute]. További információkért lásd: [valósít meg olyan biztonságos hibrid hálózati architektúra Azure][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Ajánlatok

Az Azure Active Directory végrehajtási konkrét javaslatokért tekintse meg a következő cikkeket:

- [Az Azure Active Directory tartományi szolgáltatások (AD DS) kiterjesztése][adds-extend-domain]. 
- [Telepítési útmutatója Windows Server Active Directory Azure virtuális gépeken futó][ad-azure-guidelines].

### <a name="trust"></a>Trust

A helyszíni tartományok belül egy másik erdőben a tartományok a felhőben található. Ahhoz, hogy a hitelesítés a helyszíni felhasználók a felhőben, az Azure-ban a tartományok megbízhatónak kell lennie a helyi erdő bejelentkezési tartományában. Hasonlóképpen a felhő biztosít a külső felhasználók bejelentkezési tartomány, ha szükségessé válhat a helyszíni erdő esetén, hogy bízzon meg a felhőalapú tartományt.

Az erdő szintjén által bizalmi kapcsolatok hozhatnak létre [erdőszintű megbízhatóság létrehozása][creating-forest-trusts], vagy a tartományi szinten által [külső bizalmi kapcsolatok létrehozása][creating-external-trusts]. Egy erdő szintű megbízhatósági kapcsolatot hoz létre két erdő tartományai között. Egy külső tartományi szintű megbízhatóság csak a megadott tartományok közötti kapcsolatot hoz létre. Csak a külső tartományi szintű bizalmi kapcsolatok tartomány különböző erdőkben között kell létrehozni.

Lehet, hogy egyirányú bizalmi kapcsolatok (egyirányú) vagy kétirányú (kétirányú):

* Egy egyirányú bizalmi kapcsolat lehetővé teszi, hogy a felhasználók egy tartományból vagy erdőből (néven a *bejövő* tartomány vagy erdő) egy másik fenntartott erőforrások eléréséhez (a *kimenő* tartomány vagy erdő).
* Kétirányú megbízhatósági kapcsolattal lehetővé teszi, hogy a felhasználók tartományban vagy erdőben, a másik lévő erőforrások eléréséhez.

A következő táblázat összefoglalja a megbízhatósági konfigurációi néhány egyszerű forgatókönyv:

| Eset | A helyszíni bizalmi kapcsolat | Felhő bizalmi kapcsolat |
| --- | --- | --- |
| A helyszíni felhasználóknak szükségük van a felhőben található erőforrásokhoz való hozzáférést, de nem ez fordítva is igaz |Egyirányú bejövő |Egyirányú, kimenő |
| Hozzáférést igényelnek a felhasználók a felhőben található erőforrásokhoz a helyszínen, de nem ez fordítva is igaz |Egyirányú, kimenő |Egyirányú bejövő |
| A felhasználók a felhőben és helyszíni, mind a felhőben és helyszíni fenntartott erőforrások hozzáférésre van szüksége |Kétirányú, bejövő és kimenő |Kétirányú, bejövő és kimenő |

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Active Directory automatikusan méretezhető tartományvezérlők tartozó ugyanabban a tartományban. Kérelmek egy tartomány összes tartományvezérlői különböző pontjain. Hozzáadhat egy másik tartományvezérlő, és automatikusan szinkronizál a tartományhoz. Ne konfiguráljon külön terheléselosztó át tudja irányítani a tartományvezérlők felé irányuló forgalom a tartományon belül. Győződjön meg arról, hogy minden tartományvezérlőre telepítve van-e elegendő memória- és tárolási erőforrások a tartomány-adatbázist. Ellenőrizze az összes tartományvezérlő virtuális gépek azonos méretűnek.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az egyes tartományokhoz legalább két tartományvezérlő kiépítéséhez. Ez lehetővé teszi, hogy a kiszolgálók közötti replikáció automatikus. Hozzon létre egy rendelkezésre állási készletét, a virtuális gépek minden egyes tartományhoz kezelése az Active Directory-kiszolgálóként működnek. Helyezze el legalább két kiszolgáló rendelkezésre állási csoport.

Fontolja meg is, minden tartományban egy vagy több kiszolgáló kijelölő [készenléti műveleti főkiszolgálók] [ standby-operations-masters] abban az esetben, ha meghiúsul egy fő rugalmas művelettel (FSMO) szerepkörben működő kiszolgálóhoz való csatlakozásra.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

További információ a kezelési és figyelési szempontok: [az Azure Active Directory kiterjesztése][adds-extend-domain]. 
 
További információkért lásd: [Active Directory figyelési][monitoring_ad]. Például telepítheti eszközöket [Microsoft Systems Center] [ microsoft_systems_center] ezek a feladatok végezhetők el a felügyeleti alhálózat egy felügyeleti kiszolgálón.

## <a name="security-considerations"></a>Biztonsági szempontok

Erdő szintű bizalmi kapcsolat olyan tranzitív. Egy helyi erdőben és a felhőben erdő között egy erdő szintű megbízhatósági kapcsolatot hoz létre, ha a bizalmi kapcsolat más mindkét erdőben létrehozott új tartományok az időtartam. Ha arra, hogy biztonsági okokból elválasztási tartományt használ, fontolja meg, csak a tartományi szintű bizalmi kapcsolatok létrehozása. Tartományi szintű bizalmi kapcsolatok nem tranzitív.

Active Directory-specifikus biztonsági szempontokról, tekintse meg a biztonsági szempontok című [az Azure Active Directory kiterjesztése][adds-extend-domain].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A megoldás érhető el a [GitHub] [ github] központi telepítése a referencia-architektúrában. Szüksége lesz a Powershell-parancsfájlt, amely telepíti a megoldás futtatásához az Azure parancssori felület legújabb verzióját. A referencia-architektúrában telepítéséhez kövesse az alábbi lépéseket:

1. Töltse le, vagy klónozza a megoldás mappát [GitHub] [ github] a helyi számítógépre.

2. Nyissa meg az Azure parancssori felület, és keresse meg a helyi mappát.

3. Futtassa az alábbi parancsot:
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Cserélje le `<subscription id>` az Azure-előfizetés-azonosítóval.
   
    A `<location>` paraméter esetében adjon meg egy Azure-régiót (pl. `eastus` vagy `westus`).
   
    A `<mode>` paraméter szabályozza a lépésköz legyen a központi telepítést, és a következő értékek egyike lehet:
   
   * `Onpremise`: a helyszíni szimulált környezetben telepíti.
   * `Infrastructure`: az Azure virtuális hálózat infrastruktúra és a jump mezőben telepíti.
   * `CreateVpn`: az Azure virtuális hálózati átjáró telepíti, és azt a szimulált a helyszíni hálózathoz csatlakozik.
   * `AzureADDS`: telepíti az Active Directory Tartományi kiszolgálóként működnek virtuális gépek, virtuális gépeken telepíti az Active Directory és az Azure-ban a tartomány telepíti.
   * `WebTier`: telepíti a webes réteg a virtuális gépek és a belső terheléselosztót.
   * `Prepare`: az előző központitelepítéseinek listáját, telepíti. **Ez a lehetőség ajánlott, ha Ha nem rendelkezik meglévő helyszíni hálózati, de a fent ismertetett tesztelési vagy a kiértékelési teljes referencia-architektúrában telepíteni szeretné.** 
   * `Workload`: a üzleti telepíti és adatok réteg a virtuális gépek és terheléselosztókat. Vegye figyelembe, hogy a virtuális gépeken nem szerepelnek a `Prepare` központi telepítés.

4. Várjon, amíg az üzembe helyezés befejeződik. Ha telepíti a `Prepare` központi telepítését, a több óráig tart.
     
5. A szimulált helyszíni konfiguráció használatakor konfigurálhatja a bejövő bizalmi kapcsolat:
   
   1. Az Ugrás mezőben csatlakozni (*ra-adtrust-mgmt-vm1* a a *ra-adtrust-security-rg* erőforráscsoport). Jelentkezzen be *tesztfelhasználó néven* jelszóval  *AweS0me@PW* .
   2. Ugrás a keret nyissa meg az első virtuális gép RDP-munkamenetet a *contoso.com* (a helyszíni) tartománnyal. A virtuális gép rendelkezik a 192.168.0.4 IP-címét. A felhasználónév *contoso\testuser* jelszóval  *AweS0me@PW* .
   3. Töltse le a [bejövő-trust.ps1] [ incoming-trust] parancsfájlt, és a bejövő bizalmi kapcsolat létrehozásához futtassa a *treyresearch.com* tartomány.

6. A saját helyszíni infrastruktúra használata:
   
   1. Töltse le a [bejövő-trust.ps1] [ incoming-trust] parancsfájl.
   2. Szerkesztheti a parancsprogramot, és cserélje le a a `$TrustedDomainName` változó a saját tartomány nevével.
   3. Futtassa a szkriptet.

7. A jump-box, az első virtuális gép csatlakozni a *treyresearch.com* (a felhőben) tartománnyal. A virtuális gép az IP-cím 10.0.4.4 rendelkezik. A felhasználónév *treyresearch\testuser* jelszóval  *AweS0me@PW* .

8. Töltse le a [kimenő trust.ps1] [ outgoing-trust] parancsfájlt, és a bejövő bizalmi kapcsolat létrehozásához futtassa a *treyresearch.com* tartomány. Ha a saját helyszíni gépeket használ, akkor először szerkesztheti a parancsprogramot. Állítsa be a `$TrustedDomainName` változót a helyszíni-tartománynak a nevét, és adja meg az IP-címeket ebben a tartományban az Active Directory Tartományi kiszolgálóinak a `$TrustedDomainDnsIpAddresses` változó.

9. Várjon néhány percet, amíg az előző lépéseket követve végezze el, akkor csatlakozik egy helyszíni virtuális Gépre, és a cikkben ismertetett lépésekkel [megbízhatósági kapcsolat ellenőrzése] [ verify-a-trust] meghatározásához e közötti megbízhatósági kapcsolat a *contoso.com* és *treyresearch.com* tartományok megfelelően van beállítva.

## <a name="next-steps"></a>További lépések

* A gyakorlati tanácsokat [kiterjeszti a helyszíni Active Directory tartományi szolgáltatások tartományt az Azure-bA][adds-extend-domain]
* A gyakorlati tanácsokat [egy AD FS-infrastruktúra létrehozásának] [ adfs] az Azure-ban.

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
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "biztonságos hibrid hálózati architektúra rendelkező elkülönített Active Directory-tartományok"