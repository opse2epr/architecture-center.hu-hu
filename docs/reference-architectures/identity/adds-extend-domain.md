---
title: "Az Azure Active Directory tartományi szolgáltatások (AD DS) kiterjesztése"
description: "Hogyan valósítja meg az Active Directory engedélyezési egy biztonságos hibrid hálózati architektúra az Azure-ban.\nútmutatást, vpn-átjáró, expressroute, terheléselosztó, virtuális hálózat, active Directoryval"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: azure-ad
pnp.series.next: adds-forest
ms.openlocfilehash: 7f771f77c7fa7f266dcce9f5b45e5be658213b8d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="extend-active-directory-domain-services-ad-ds-to-azure"></a>Az Azure Active Directory tartományi szolgáltatások (AD DS) kiterjesztése

A referencia-architektúrában bemutatja, hogyan terjeszthető ki az Active Directory környezet az Azure-bA elosztott hitelesítési szolgáltatásokat használó [Active Directory tartományi szolgáltatások (AD DS)][active-directory-domain-services].  [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

[![0]][0] 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

Active Directory tartományi szolgáltatások felhasználói, számítógép, alkalmazás vagy többi egy biztonsági tartomány szereplő identitások hitelesítésére szolgál. Lehet üzemeltethető a helyszínen, de ha az alkalmazás üzemel részben a helyszíni és részben az Azure-ban Ez a funkció az Azure-ban replikálásához hatékonyabb lehet. Ez csökkentheti a várakozási okozta hitelesítési küld, és fut a helyszíni AD DS biztonsági helyi engedélyezési kérelmeket a felhőből. 

Ez az architektúra gyakran használják VPN- vagy ExpressRoute kapcsolattal csatlakoztatott a helyszíni hálózat és az Azure virtuális hálózat. Ez az architektúra is támogatja a kétirányú replikációt, azaz a módosítás helyi vagy a felhőben, és mindkét források konzisztens megmarad. Ez az architektúra a gyakori felhasználási, amelyben funkciók a helyszíni és Azure-ban, és az alkalmazások és szolgáltatásokról, amelyek a hitelesítés elvégzésére Active Directory használatával közötti elosztott hibrid alkalmazások közé tartoznak.

További szempontokat lásd: [megoldás választása a integrálása a helyszíni Active Directoryról szinkronizálva az Azure][considerations]. 

## <a name="architecture"></a>Architektúra 

Ez az architektúra kibővíti az architektúra látható [Azure és az Internet között DMZ][implementing-a-secure-hybrid-network-architecture-with-internet-access]. A következő összetevőket tartalmaz.

* **A helyszíni hálózat**. A helyszíni hálózat helyi Active Directory-kiszolgálók által végrehajtható műveleteket hitelesítési és engedélyezési helyszíni található összetevőket tartalmazza.
* **Active Directory-kiszolgálók**. Ezek a tartományvezérlők végrehajtási a directory services (AD DS) futtató virtuális gépek a felhőben. Ezek a kiszolgálók az Azure virtuális hálózatban futtat összetevők is hitelesítésre.
* **Active Directory-alhálózathoz**. Az Active Directory tartományi szolgáltatások kiszolgálókon tárolt külön alhálózathoz. Hálózati biztonsági csoport (NSG) szabályok az Active Directory tartományi szolgáltatások kiszolgálók védelméhez, és adja meg a tűzfal váratlan forrásból ellen.
* **Azure átjáró és az Active Directory-szinkronizálás**. Az Azure átjáró biztosítja a kapcsolatot a helyszíni hálózat és az Azure VNet között. Ez lehet egy [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute]. Az Active Directory-kiszolgálók, a felhőben és helyszíni közötti összes szinkronizálási kérelem továbbítása az átjárót. Felhasználó által definiált útvonalak (udr-EK) kezeli a helyszíni forgalomhoz, amely az Azure-bA útválasztást. Az Active Directory-kiszolgálók érkező vagy oda irányuló forgalom nem halad át a hálózati virtuális készülékek (NVAs) lehet használni.

Udr-EK és a NVAs konfigurálásával kapcsolatos további információkért lásd: [valósít meg olyan biztonságos hibrid hálózati architektúra Azure][implementing-a-secure-hybrid-network-architecture]. 

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Határozza meg a [Virtuálisgép-méretet] [ vm-windows-sizes] követelmények, a hitelesítési kérelmek várható mennyisége alapján. Használja a kiindulási pontként a helyszíni AD DS futtató gépek előírásainak, és egyezik meg azokat az Azure Virtuálisgép-méretek. Központi telepítésének végrehajtása után figyelni kihasználtságát és a skála felfelé vagy lefelé a tényleges betöltést a virtuális gépeken alapul. Active Directory tartományi szolgáltatások tartományvezérlő méretezéssel kapcsolatos további információkért lásd: [kapacitástervezés Active Directory tartományi szolgáltatások][capacity-planning-for-adds].

Hozzon létre egy külön virtuális adatlemez az adatbázis, a naplókat, és a SYSVOL tárolására az Active Directory. Az operációs rendszer ugyanazon a lemezen tárolja ezeket az elemeket. Vegye figyelembe, hogy alapértelmezés szerint egy virtuális géphez csatolt adatok lemezt használjon író gyorsítótárazást. Azonban az űrlap gyorsítótárazásának ütközhetnek Active Directory tartományi szolgáltatások követelményeinek. Emiatt, állítsa be a *állomás gyorsítótár beállításokat szabályozó* beállítása a adatlemez *nincs*. További információkért lásd: [a Windows Server AD DS-adatbázis és a SYSVOL helyét][adds-data-disks].

Legalább két virtuális gépeket futtató tartományvezérlők Active Directory tartományi szolgáltatások telepítéséhez, és hozzáadhatja őket a egy [rendelkezésre állási csoport][availability-set].

### <a name="networking-recommendations"></a>Hálózatokra vonatkozó javaslatok

Konfigurálja a virtuális gép hálózati kártya (NIC) Active Directory tartományi szolgáltatások kiszolgálónként teljes tartománynevek szolgáltatási (DNS) támogatása statikus magánhálózati IP-cím. További információkért lásd: [hozzá statikus magánhálózati IP-cím beállítása az Azure portálon][set-a-static-ip-address].

> [!NOTE]
> Bármely AD DS-ben a virtuális gép hálózati Adapternek nem konfigurál egy nyilvános IP-címet. Lásd: [biztonsági szempontok] [ security-considerations] további részleteket.
> 
> 

Az Active Directory-alhálózathoz NSG bejövő forgalom engedélyezése a helyszíni szabályok igényel. Részletes információk az Active Directory tartományi szolgáltatások által használt portokon: [Active Directory és az Active Directory Domain Services Port Requirements][ad-ds-ports]. Gondoskodjon arról is, a UDR táblák nem irányíthatja a NVAs ebben az architektúrában használt Active Directory tartományi szolgáltatások forgalmát. 

### <a name="active-directory-site"></a>Active Directory-hely

Az AD DS-ben a hely a fizikai hely, hálózati vagy eszközök jelöli. Az AD DS-helyek Active Directory tartományi szolgáltatások adatbázis-replikáció kezelésére AD DS-objektum, amely közel egy másik találhatók, és a nagy sebességű hálózati csatlakoztatott csoportosításával szolgálnak. Active Directory tartományi szolgáltatások jelölje be az AD DS adatbázis, a helyek közötti replacating legjobb stratégiája logikát tartalmaz.

Azt javasoljuk, hogy hozzon létre egy AD DS-helyen, beleértve a az alkalmazás az Azure-ban meghatározott alhálózatokat. Ezt követően konfigurálja a helykapcsolatot közé a helyszíni AD DS-helyek és az Active Directory tartományi szolgáltatások automatikusan végrehajtják a lehető leghatékonyabb adatbázis-replikáció. Vegye figyelembe, hogy az adatbázis-replikáció a kezdeti konfiguráció túl kevés igényel.

### <a name="active-directory-operations-masters"></a>Active Directory műveleti főkiszolgálók

A műveleti főkiszolgáló szerepkör rendelhet az Active Directory tartományi szolgáltatások tartományvezérlő támogatja a konzisztencia-ellenőrzés az Active Directory tartományi szolgáltatások replikációs adatbázisok példányai között. Öt műveleti főkiszolgálói szerepkörök állnak rendelkezésre: séma-főkiszolgáló, a tartománynév-kiosztási főkiszolgálót, a relatív azonosítók főkiszolgálója, a elsődleges tartományvezérlő fő emulátor és a infrastruktúra-főkiszolgálóhoz. Ezek a szerepkörök kapcsolatos további információkért lásd: [Mik azok a műveleti főkiszolgálók?] [ad-ds-operations-masters].

Azt javasoljuk, hogy nem rendel műveleti főkiszolgálói szerepkörökre az Azure szolgáltatásba telepített tartományvezérlőre.

### <a name="monitoring"></a>Figyelés

A tartományvezérlő virtuális gépeket, valamint az AD DS szolgáltatások erőforrások figyelése, és hozzon létre egy csomagot, hogy gyorsan javítsa az esetleges problémákat. További információkért lásd: [Active Directory figyelési][monitoring_ad]. Eszközök is telepítheti, mint [Microsoft Systems Center] [ microsoft_systems_center] a felügyeleti kiszolgálón (lásd az architektúra diagramja) segítségével a következő feladatok végrehajtására.  

## <a name="scalability-considerations"></a>Méretezési szempontok

Active Directory tartományi szolgáltatások célja, a méretezhetőség érdekében. Nem kell terheléselosztót vagy az Active Directory tartományi szolgáltatások tartományvezérlő kéréseiket forgalom vezérlő konfigurálása. A csak méretezhetőség szempont, hogy az Adatbetöltési követelményeinek futó Active Directory tartományi szolgáltatások a hálózat megfelelő méretűek a virtuális gép konfigurálásához, a figyelheti a virtuális gépek és a skála felfelé vagy lefelé szükség szerint terhelését.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A az AD DS-ben futó virtuális gépeket telepíteni egy [rendelkezésre állási csoport][availability-set]. Fontolja meg is, a szerepkör hozzárendelése [készenléti műveleti főkiszolgáló] [ standby-operations-masters] legalább egy kiszolgálót, és esetleg a követelményeitől függően több. A készenléti műveleti főkiszolgáló egy aktív másolatot készít a műveleti főkiszolgáló, amely a feladatátvétel során főkiszolgálók kiszolgáló elsődleges műveletek helyett használható.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Rendszeresen biztonsági másolatokat Active Directory tartományi Szolgáltatásokban. Nem egyszerűen másolása a VHD-fájlok helyett rendszeres biztonsági mentéseket a tartományvezérlők, mert az Active Directory tartományi szolgáltatások adatbázis-fájl a virtuális merevlemez nem, előfordulhat, konzisztens állapotban másolását követően lehetetlenné indítsa újra az adatbázist.

Nem állnak le egy tartományvezérlő virtuális Gépnek az Azure portál használatával. Ehelyett állítsa le és indítsa újra a vendég operációs rendszerből. Címtárszolgáltatás le a portálon keresztül hatására a virtuális gép felszabadítása, amely mindkét alaphelyzetbe állítja a `VM-GenerationID` és a `invocationID` az Active Directory-tárház. Ez az Active Directory tartományi szolgáltatások relatív azonosító (RID) készlet elveti jelöli meg a SYSVOL nem mérvadó, és előfordulhat, hogy a tartományvezérlő újrakonfigurálása.

## <a name="security-considerations"></a>Biztonsági szempontok

AD DS-kiszolgálók hitelesítési szolgáltatásokat nyújtanak, és egy vonzó célhoz támadások. A biztonságos, megelőzése közvetlen internetkapcsolattal helyezi el az Active Directory tartományi szolgáltatások kiszolgálók külön alhálózathoz van egy tűzfal működött egy NSG. Zárja be az Active Directory tartományi szolgáltatások kiszolgálók szükséges hitelesítési, engedélyezési és szinkronizálási kiszolgáló kivételével minden port. További információkért lásd: [Active Directory és az Active Directory Domain Services Port Requirements][ad-ds-ports].

Vegye fontolóra egy további biztonsági szegélyhálózati körül alhálózatokat és NVAs, két kiszolgálók leírtak [valósít meg olyan biztonságos hibrid hálózati architektúra Internet-hozzáféréssel rendelkező Azure] [ implementing-a-secure-hybrid-network-architecture-with-internet-access].

A BitLocker vagy a Azure lemeztitkosítás segítségével titkosíthatja a lemezt az AD DS-adatbázis tárolásához.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez

A megoldás érhető el a [Github] [ github] központi telepítése a referencia-architektúrában. Szüksége lesz a legújabb verzióját a [Azure CLI] [ azure-powershell] futtatni a Powershell-parancsfájlt, amely a megoldás telepít. A referencia-architektúrában telepítéséhez kövesse az alábbi lépéseket:

1. Töltse le, vagy klónozza a megoldás mappát [Github] [ github] a helyi számítógépre.

2. Nyissa meg az Azure parancssori felület, és keresse meg a helyi mappát.

3. Futtassa az alábbi parancsot:
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
    Cserélje le `<subscription id>` az Azure-előfizetés-azonosítóval.
    A `<location>`, adjon meg egy Azure-régió, például `eastus` vagy `westus`.
    A `<mode>` paraméter szabályozza a lépésköz legyen a központi telepítést, és a következő értékek egyike lehet:
    * `Onpremise`: a helyszíni szimulált környezetben telepíti.
    * `Infrastructure`: az Azure virtuális hálózat infrastruktúra és a jump mezőben telepíti.
    * `CreateVpn`: az Azure virtuális hálózati átjáró telepíti, és azt a szimulált a helyszíni hálózathoz csatlakozik.
    * `AzureADDS`: telepíti az Active Directory tartományi szolgáltatások kiszolgálóként működnek virtuális gépek, virtuális gépeken telepíti az Active Directory és az Azure-ban a tartomány telepíti.
    * `Workload`: a nyilvános és titkos DMZ-k és a munkaterhelés szint telepíti.
    * `All`: az előző központitelepítéseinek listáját, telepíti. **Ez a lehetőség ajánlott, ha Ha nem rendelkezik meglévő helyszíni hálózati, de a fent ismertetett tesztelési vagy a kiértékelési teljes referencia-architektúrában telepíteni szeretné.**

4. Várjon, amíg az üzembe helyezés befejeződik. Ha telepíti a `All` központi telepítését, a több óráig tart.

## <a name="next-steps"></a>Következő lépések

* A gyakorlati tanácsokat [létrehozása az Active Directory tartományi szolgáltatások Erőforráserdő] [ adds-resource-forest] az Azure-ban.
* A gyakorlati tanácsokat [egy Active Directory összevonási szolgáltatások (AD FS) infrastruktúra létrehozásának] [ adfs] az Azure-ban.

<!-- links -->
[adds-resource-forest]: adds-forest.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[adds-data-disks]: https://msdn.microsoft.com/library/azure/jj156090.aspx#BKMK_PlaceDB
[ad-ds-operations-masters]: https://technet.microsoft.com/library/cc779716(v=ws.10).aspx
[ad-ds-ports]: https://technet.microsoft.com/library/dd772723(v=ws.11).aspx
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-powershell]: /powershell/azureps-cmdlets-docs
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[capacity-planning-for-adds]: http://social.technet.microsoft.com/wiki/contents/articles/14355.capacity-planning-for-active-directory-domain-services.aspx
[considerations]: ./considerations.md
[GitHub]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-extend-domain
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[security-considerations]: #security-considerations
[set-a-static-ip-address]: https://azure.microsoft.com/documentation/articles/virtual-networks-static-private-ip-arm-pportal/
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[vm-windows-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes

[0]: ./images/adds-extend-domain.png "biztonságos és az Active Directory hibrid hálózati architektúra"
