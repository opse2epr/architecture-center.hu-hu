---
title: AD DS-erőforráserdő létrehozása az Azure-ban
titleSuffix: Azure Reference Architectures
description: >-
  Megbízható Active Directory-tartomány létrehozása az Azure-ban.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 05/02/2018
ms.custom: seodec18
ms.openlocfilehash: 3e3c3c8ff12bab85a96d4eb879f81195d22e79f8
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/20/2018
ms.locfileid: "53643781"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Active Directory Domain Services- (AD DS-) erőforráserdő létrehozása az Azure-ban

A referenciaarchitektúra bemutatja, hogyan hozható létre egy külön Active Directory-tartomány az Azure-ban, amelyet a helyszíni AD-erdőben található tartományok megbízhatónak tekintenek. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Biztonságos hibrid hálózati architektúra külön Active Directory-tartományok](./images/adds-forest.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az Active Directory Domain Services (AD DS) hierarchikus struktúrában tárolja az identitásadatokat. A hierarchikus struktúra legfelső szintű csomópontját erdőnek hívják. Az erdő tartományokat, a tartományok pedig egyéb típusú objektumokat tartalmaznak. Ez a referenciaarchitektúra egy AD DS-erdőt hoz létre az Azure-ban, amely egyirányú kimenő bizalmi kapcsolattal rendelkezik egy helyszíni tartománnyal. Az Azure-ban lévő erdő tartalmaz egy olyan tartományt, amely a helyszínen nem létezik. A bizalmi kapcsolat miatt a helyszíni tartományokra történő bejelentkezések megbízhatók a külön Azure-tartományban lévő erőforrások eléréséhez.

Az architektúra gyakori használati módjai például a felhőben tárolt objektumok és identitások biztonsági elkülönítésének fenntartása, illetve az egyes tartományok migrálása a helyszínről a felhőbe.

További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations]. 

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőket tartalmazza.

- **Helyszíni hálózat**. A helyszíni hálózat a saját Active Directory-erdőjét és tartományait tartalmazza.
- **Active Directory-kiszolgálók**. Ezek a felhőben virtuális gépként futó tartományvezérlők, amelyek tartományi szolgáltatásokat biztosítanak. Ezek a kiszolgálók egy egy vagy több tartományt tartalmazó erdőt üzemeltetnek a helyszíniektől elkülönítve.
- **Egyirányú bizalmi kapcsolat**. Az ábrán az Azure-ban lévő tartományból a helyszíni tartományba tartó egyirányú bizalmi kapcsolatra látható egy példa. Ez a kapcsolat lehetővé teszi a helyszíni felhasználók számára, hogy elérjék az Azure-ban lévő tartomány erőforrásait, de fordított irányban ez nem lehetséges. Kétirányú megbízhatósági kapcsolat is létrehozható, ha a felhőalapú felhasználóknak is hozzáférést kell biztosítani a helyszíni erőforrásokhoz.
- **Active Directory-alhálózat**. Az AD DS-kiszolgálók külön alhálózaton üzemelnek. A hálózati biztonsági csoport (NSG) szabályai megvédik az AD DS-kiszolgálókat, és tűzfalat biztosítanak a nem várt forrásból érkező adatforgalom ellen.
- **Azure-átjáró**. Az Azure-átjáró kapcsolatot biztosít a helyszíni hálózat és az Azure-beli virtuális hálózat között. Ez lehet [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute]. További információ: [Biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban][implementing-a-secure-hybrid-network-architecture].

## <a name="recommendations"></a>Javaslatok

Az Active Directory Azure-ban való megvalósítására vonatkozó konkrét ajánlásokért tekintse meg az alábbi cikkeket:

- [Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][adds-extend-domain].
- [Útmutató a Windows Server Active Directory szolgáltatás Azure-beli virtuális gépekre való telepítéséhez][ad-azure-guidelines].

### <a name="trust"></a>Bizalmi kapcsolat

A helyszíni tartományokat nem ugyanaz az erdő tartalmazza, mint a felhőben lévőket. A helyszíni felhasználók felhőbeli hitelesítésének engedélyezéséhez az Azure-ban lévő tartományoknak meg kell bízniuk a helyszíni erdőben található bejelentkezési tartományban. Hasonlóképpen, ha a felhő bejelentkezési tartományt biztosít a külső felhasználók számára, szükség lehet arra, hogy a helyszíni erdő megbízzon a felhőtartományban.

Bizalmi kapcsolatokat létesíthet erdőszinten [erdőszintű bizalmi kapcsolatok létrehozásával][creating-forest-trusts] vagy tartományszinten [külső bizalmi kapcsolatok létrehozásával][creating-external-trusts]. Az erdőszintű bizalmi kapcsolat két erdő összes tartománya között kapcsolatot hoz létre. A külső tartományszintű bizalmi kapcsolat csak két adott tartomány között hoz létre kapcsolatot. Különböző erdőkben lévő tartományok között csak külső tartományszintű bizalmi kapcsolatot hozzon létre.

A bizalmi kapcsolatok lehetnek egyirányúak vagy kétirányúak:

- Az egyirányú bizalmi kapcsolat lehetővé teszi egy tartományban vagy erdőben (*bejövő* tartomány vagy erdő) lévő felhasználóknak, hogy hozzáférjenek a másikban (*kimenő* tartomány vagy erdő) tárolt erőforrásokhoz.
- A kétirányú bizalmi kapcsolat lehetővé teszi a tartományban vagy az erdőben lévő felhasználók számára, hogy hozzáférjenek a másikban tárolt erőforrásokhoz.

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

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github]. Vegye figyelembe, hogy a teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza a VPN gateway létrehozása, és futtatja a parancsfájlokat, amelyek az AD DS konfigurálása.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy töltse le a zip-fájlját a [GitHub-adattár](https://github.com/mspnp/identity-reference-architectures).

2. Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

3. Telepítse a [Azure építőelemeiről](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm-csomag.

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált helyszíni adatközpont üzembe helyezése

1. Keresse meg a `identity/adds-forest` mappájában, a GitHub-adattárban.

2. Nyissa meg az `onprem.json` fájlt. Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.

3. Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Az Azure virtuális hálózat üzembe helyezése

1. Nyissa meg az `azure.json` fájlt. Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.

2. Ugyanebben a fájlban keresse meg a példányok `sharedKey` , és adja meg a VPN-kapcsolat megosztott kulcsok.

    ```json
    "sharedKey": "",
    ```

3. Futtassa a következő parancsot, és várjon, amíg az üzembe helyezés befejeződik.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   Telepítse a helyszíni virtuális hálózatnak ugyanabban az erőforráscsoportban.

### <a name="test-the-ad-trust-relation"></a>Az AD megbízhatósági kapcsolat tesztelése

1. Az Azure Portallal, keresse meg a létrehozott erőforráscsoportot.

2. Az Azure portal használatával keresse meg a virtuális gép nevű `ra-adt-mgmt-vm1`.

3. Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához. A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `onprem.json` alkalmazásparaméter-fájlt.

4. A belül a távoli asztali munkamenetet, nyissa meg egy másik távoli asztali munkamenetet 192.168.0.4, amely az IP-címet a virtuális gép nevű `ra-adtrust-onpremise-ad-vm1`. A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `azure.json` alkalmazásparaméter-fájlt.

5. A belül a távoli asztali munkamenetet `ra-adtrust-onpremise-ad-vm1`, lépjen a **Kiszolgálókezelő** kattintson **eszközök** > **Active Directory-tartományok és megbízhatósági kapcsolatok**.

6. A bal oldali ablaktáblán kattintson a jobb gombbal a contoso.com, és válassza **tulajdonságok**.

7. Kattintson a **megbízik** fülre. Egy bejövő bizalmi kapcsolat állapottal treyresearch.net kell megjelennie.

![Képernyőkép az Active Directory erdőszintű bizalmi kapcsolat párbeszédpanel](./images/ad-forest-trust.png)

## <a name="next-steps"></a>További lépések

- A [helyszíni AD DS-tartomány Azure-ra való kiterjesztésére][adds-extend-domain] vonatkozó ajánlott eljárások megismerése
- Az Azure-alapú [AD FS-infrastruktúra létrehozására][adfs] vonatkozó ajánlott eljárások megismerése.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://microsoft.com/cloud-platform/system-center
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
