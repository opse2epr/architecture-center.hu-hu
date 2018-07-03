---
title: Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra
description: A helyszíni Active Directory-tartomány kiterjesztése az Azure-bA
author: telmosampaio
ms.date: 05/02/2018
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: ecf24a05d071c0d0283fc962b13285108b5ac4bd
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142273"
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a>Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra

Ez a referenciaarchitektúra bemutatja, hogyan terjeszthető ki a az Active Directory-környezetet az Azure Active Directory Domain Services (AD DS) használatával, elosztott hitelesítési szolgáltatások biztosításához. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az AD DS egy biztonsági tartományban található felhasználó, számítógép, alkalmazás és egyéb identitások hitelesítésére használható. Az AD DS üzemeltethető a helyszínen, de ha az alkalmazás részben a helyszínen, részben pedig az Azure-ban üzemel, a funkció Azure-ban való replikálása hatékonyabb lehet. Ez csökkentheti a késést, amelyet a hitelesítési és a helyi engedélyezési kérelmek a felhőből a helyszínen futó AD DS-be való visszaküldése okoz. 

Ezt az architektúrát gyakran használják, ha a helyszíni hálózatot és az Azure-beli virtuális hálózatot VPN- vagy ExpressRoute-kapcsolat köti össze. Ez az architektúra a kétirányú replikációt is támogatja, ez azt jelenti, hogy a módosítások a helyszínen vagy a felhőben is elvégezhetők, és mindkét forrás egységes lesz. Az architektúra gyakori használati módjai közé tartoznak a hibrid alkalmazások, amelyekben a funkciók megoszlanak a helyszíni hely és az Azure között, valamint azok az alkalmazások és szolgáltatások, amelyek az Active Directoryval végzik el a hitelesítést.

További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations]. 

## <a name="architecture"></a>Architektúra 

Ez az architektúra a [Szegélyhálózat (DMZ) az Azure és az internet között][implementing-a-secure-hybrid-network-architecture-with-internet-access] című részben használt architektúrát bővíti ki. A következő összetevőket tartalmazza.

* **Helyszíni hálózat**. A helyszíni hálózat tartalmazza azokat a helyi Active Directory-kiszolgálókat, amelyek a helyszíni összetevők hitelesítését és engedélyezését végezhetik el.
* **Active Directory-kiszolgálók**. Ezek a felhőben virtuális gépként futó tartományvezérlők, amelyek címtárszolgáltatásokat (AD DS) biztosítanak. Ezek a kiszolgálók az Azure-beli virtuális hálózaton futó összetevők hitelesítését végezhetik el.
* **Active Directory-alhálózat**. Az AD DS-kiszolgálók külön alhálózaton üzemelnek. A hálózati biztonsági csoport (NSG) szabályai megvédik az AD DS-kiszolgálókat, és tűzfalat biztosítanak a nem várt forrásból érkező adatforgalom ellen.
* **Azure Gateway és Active Directory szinkronizálása**. Az Azure-átjáró kapcsolatot biztosít a helyszíni hálózat és az Azure-beli virtuális hálózat között. Ez lehet [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute]. A felhőben és a helyszínen lévő Active Directory-kiszolgálók közötti minden forgalom ezen az átjárón halad keresztül. A felhasználó által megadott útvonalak (UDR-ek) kezelik az Azure-ba tartó helyszíni forgalom útválasztását. Az Active Directory-kiszolgálók bemenő és kimenő forgalma nem halad át az ebben a forgatókönyvben használt hálózati virtuális berendezéseken (NVA-kon).

További információ az UDR-ek és az NVA-k konfigurálásáról: [Biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban][implementing-a-secure-hybrid-network-architecture]. 

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Határozza meg a [virtuális gép méretkövetelményeit][vm-windows-sizes] a hitelesítési kérelmek várt mérete alapján. Kiindulópontként használja az AD DS-t helyszínen üzemeltető gépek specifikációit, és egyeztesse őket az Azure-beli virtuális gépek méreteivel. Telepítés után monitorozza a kihasználtságot, és a virtuális gép tényleges terhelése alapján vertikálisan skálázza fel vagy le a kapacitást. További információ az AD DS-tartományvezérlők méretezéséről: [Az Active Directory Domain Services kapacitásának tervezése][capacity-planning-for-adds].

Hozzon létre egy külön virtuális adatlemezt az Active Directoryhoz tartozó adatbázis, naplók és SYSVOL tárolásához. Ezeket ne ugyanazon a lemezen tárolja, amelyen az operációs rendszer van. Vegye figyelembe, hogy a virtuális géphez csatolt adatlemezek alapértelmezés szerint visszaírt gyorsítótárazást használnak. A gyorsítótárazás ezen formája azonban ütközhet az AD DS követelményeivel. Ezért az adatlemez *Host Cache Preference* (Gazdagép gyorsítótár-beállítása) beállítást állítsa *None* (Nincs) értékre. További információ: [A Windows Server AD DS-adatbázis és SYSVOL elhelyezése][adds-data-disks].

Helyezzen üzembe tartományvezérlőként legalább két, AD DS-t futtató virtuális gépet, és adja hozzá őket egy [rendelkezésre állási csoporthoz][availability-set].

### <a name="networking-recommendations"></a>Hálózatokra vonatkozó javaslatok

Konfigurálja minden AD DS-kiszolgáló virtuális gép hálózati adaptererét (NIC) konfigurálja egy statikus magánhálózati IP-címmel a tartománynév-szolgáltatás (DNS) teljes körű támogatása érdekében. További információ: [Statikus magánhálózati IP-cím beállítása az Azure Portalon][set-a-static-ip-address].

> [!NOTE]
> Ne konfigurálja az AD DS-kiszolgálók virtuális gép hálózati adapterét nyilvános IP-címmel. További részleteket a [Biztonsági szempontok][security-considerations] című részben talál.
> 
> 

Az Active Directory alhálózati biztonsági csoporthoz olyan szabályok szükségesek, amelyek engedélyezik a helyszínről bejövő forgalmat. Részletes információ az AD DS által használt portokról: [Az Active Directory és az Active Directory Domain Services portokra vonatkozó követelményei][ad-ds-ports]. Győződjön meg arról is, hogy az UDR-táblák nem irányítják az AD DS-forgalmat az architektúrában használt NVA-kra. 

### <a name="active-directory-site"></a>Active Directory-hely

Az AD DS-ben a hely egy fizikai helyet, hálózatot vagy eszközgyűjteményt jelent. Az AD DS-helyek az AD DS adatbázis-replikációk kezelésére használhatók az egymáshoz közel elhelyezkedő és nagy sebességű hálózattal összekapcsolt AD DS-objektumok csoportosításával. Az AD DS beépített logikája képes kiválasztani az AD DS-adatbázisok helyek közötti replikációlásának legjobb stratégiáját.

Azt javasoljuk, hogy hozzon létre egy AD DS-helyet, amely tartalmazza az Azure-ban lévő alkalmazáshoz megadott alhálózatokat. Ezután konfiguráljon egy helykapcsolatot a helyszíni AD DS-helyek között, és az AD DS automatikusan elvégzi a lehető leghatékonyabb adatbázis-replikációt. Vegye figyelembe, hogy ez az adatbázis-replikáció a kezdeti konfigurációnál nem sokkal igényel többet.

### <a name="active-directory-operations-masters"></a>Active Directory műveleti főkiszolgálók

A műveleti főkiszolgálói szerepkör hozzárendelhető az AD DS-tartományvezérlőkhöz, így biztosítható a replikált AD DS-adatbázisok példányai közötti konzisztencia-ellenőrzés támogatása. Öt műveleti főkiszolgálói szerepkör van: sémakezelő főkiszolgáló, tartománynév-kiosztási főkiszolgáló, relatívazonosító-kezelő főkiszolgáló, elsődleges tartományvezérlő-főkiszolgáló emulátor, infrastruktúrakezelő főkiszolgáló. További információ ezekről a szerepkörökről: [Mik a műveleti főkiszolgálók?][ad-ds-operations-masters].

Azt javasoljuk, hogy ne rendeljen főkiszolgálói szerepköröket az Azure-ban üzembe helyezett tartományvezérlőkhöz.

### <a name="monitoring"></a>Figyelés

Monitorozza a tartományvezérlő virtuális gépek, valamint az AD DS Services erőforrásait, és hozzon létre egy tervet az esetleges problémák gyors javítására. További információ: [Az Active Directory monitorozása][monitoring_ad]. A feladatok megkönnyítéséhez olyan eszközöket is telepíthet a monitorozási kiszolgálóra (lásd az architektúra diagramját), mint a [Microsoft Systems Center][microsoft_systems_center].  

## <a name="scalability-considerations"></a>Méretezési szempontok

Az AD DS-t méretezhetőségre tervezték. Nem kell terheléselosztót vagy forgalomirányítót konfigurálnia a kérelmek AD DS-tartományvezérlőkre történő irányításához. Az egyetlen méretezhetőségi szempont az AD DS-t futtató virtuális gépek konfigurálása a hálózati terhelési követelményeinek megfelelően, a virtuális gépek terhelésének monitorozása, valamint a szükség szerinti vertikális fel- vagy leskálázás.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az AD DS-t futtató virtuális gépeket egy [rendelkezésre állási csoportban][availability-set] helyezze üzembe. Fontolja meg azt is, hogy a [készenléti műveleti főkiszolgálói][standby-operations-masters] szerepkört legalább egy, vagy lehetőség szerint több kiszolgálóhoz is hozzárendeli, a követelményektől függően. A készenléti műveleti főkiszolgáló a műveleti főkiszolgáló egy aktív másolata, amely az elsődleges műveleti főkiszolgáló helyett használható feladatátvételkor.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Rendszeresen készítsen biztonsági mentést az AD DS-ről. A tartományvezérlők VHD-fájljait másolása nem elegendő a rendszeres biztonsági mentés helyett, mert előfordulhat, hogy a virtuális lemezen lévő AD DS-adatbázisfájlok nincsenek konzisztens állapotban a másoláskor, ezért az adatbázist nem lehet újraindítani.

Ne állítsa le a tartományvezérlő virtuális gépeket az Azure Portalról. Ehelyett a vendég operációs rendszerből állítsa le és indítsa újra őket. A portálon keresztüli leállítás a virtuális gép felszabadítását okozza, amely alaphelyzetbe állítja az Active Directory-adattár `VM-GenerationID` és `invocationID` beállításait. Ez elveti az AD DS relatív azonosítójának (RID) készletét, nem mérvadóként jelöli meg a SYSVOL-t, és szükségessé teheti a tartományvezérlő újrakonfigurálását.

## <a name="security-considerations"></a>Biztonsági szempontok

Az AD DS-kiszolgálók hitelesítési szolgáltatásokat nyújtanak, és a támadások vonzó célpontjai. A védelmük érdekében akadályozza meg a közvetlen internetkapcsolatot úgy, hogy az AD DS-kiszolgálókat egy külön alhálózatra helyezi, ahol egy hálózati biztonsági csoport tűzfalként szolgál. Zárjon be minden portot az AD DS-kiszolgálókon, kivéve a hitelesítéshez, engedélyezéshez és kiszolgálószinkronizáláshoz szükséges portokat. További információ lásd: [Az Active Directory és az Active Directory Domain Services portjainak követelményei][ad-ds-ports].

Fontolja meg egy kiegészítő biztonsági terület létrehozását is a kiszolgálók körül két alhálózat és hálózati virtuális készülékek (NVA-k) használatával a [biztonságos, internet-hozzáféréssel rendelkező hibrid hálózati architektúra Azure-ban való megvalósítását][implementing-a-secure-hybrid-network-architecture-with-internet-access] ismertető dokumentumban leírtak szerint.

A BitLocker vagy az Azure Disk Encryption használatával titkosítsa az AD DS-adatbázist üzemeltető lemezt.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github]. Vegye figyelembe, hogy a teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza a VPN gateway létrehozása, és futtatja a parancsfájlokat, amelyek az AD DS konfigurálása.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált helyszíni adatközpont üzembe helyezése

1. Keresse meg a `identity/adds-extend-domain` mappájában, a GitHub-adattárban.

2. Nyissa meg az `onprem.json` fájlt. Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.

3. Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Az Azure virtuális hálózat üzembe helyezése

1. Nyissa meg az `azure.json` fájlt.  Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait. 

2. Ugyanebben a fájlban keresse meg a példányok `sharedKey` , és adja meg a VPN-kapcsolat megosztott kulcsok. 

    ```bash
    "sharedKey": "",
    ```

3. Futtassa a következő parancsot, és várjon, amíg az üzembe helyezés befejeződik.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   Telepítse a helyszíni virtuális hálózatnak ugyanabban az erőforráscsoportban.

### <a name="test-connectivity-with-the-azure-vnet"></a>Az Azure virtuális hálózatok közötti kapcsolat tesztelése

Üzembe helyezés befejezése után tesztelheti conectivity a szimulált helyszíni környezetből az Azure vnetre.

1. Az Azure Portallal, keresse meg a létrehozott erőforráscsoportot.

2. Keresse meg a virtuális gép nevű `ra-onpremise-mgmt-vm1`.

3. Kattintson a `Connect` a virtuális géphez távoli asztali kapcsolat megnyitásához. A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `onprem.json` alkalmazásparaméter-fájlt.

4. A belül a távoli asztali munkamenetet, nyissa meg egy másik távoli asztali munkamenetet címe: 10.0.4.4, amely az IP-címet a virtuális gép nevű `adds-vm1`. A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `azure.json` alkalmazásparaméter-fájlt.

5. A belül a távoli asztali munkamenetet `adds-vm1`, lépjen a **Kiszolgálókezelő** kattintson **további kezelendő kiszolgálók hozzáadása.** 

6. Az a **Active Directory** lapra, majd **Keresés most**. Megjelenik egy listáját az Active Directory, AD DS és a webes virtuális gépeket.

   ![](./images/add-servers-dialog.png)

## <a name="next-steps"></a>További lépések

* Az Azure-beli [AD DS-erőforráserdő létrehozására][adds-resource-forest] vonatkozó ajánlott eljárások megismerése.
* Az Azure-alapú [Active Directory Federation Services- (AD FS-) infrastruktúra létrehozására][adfs] vonatkozó ajánlott eljárások megismerése.

<!-- links -->

[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: /azure/virtual-network/virtual-networks-static-private-ip-arm-pportal
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "Biztonságos hibrid hálózati architektúra az Active Directoryval"
