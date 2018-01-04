---
title: "Egy Windows virtuális gép futtatása az Azure-on"
description: "A Windows virtuális gépek Azure, a méretezhetőséget, a rugalmasság, a kezelhetőségi és a biztonsági kifizető figyelmet futtatásának módját."
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 71eeebae1f557ecbb6f33c4a7e37a278204f3dcd
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/02/2018
---
# <a name="run-a-windows-vm-on-azure"></a>Egy Windows virtuális gép futtatása az Azure-on

A referencia-architektúrában jeleníti meg az Azure-on futó Windows virtuális gépek (VM) bevált gyakorlatok csoportja. Ez magában foglalja a hálózati és tárolási összetevők mellett a virtuális gép üzembe helyezéséhez javaslatok. Ez az architektúra egyetlen Virtuálisgép-példány futtatása is használható, és alapjául szolgáló összetettebb architektúrák például N szintű alkalmazásokhoz. [**A megoldás üzembe helyezéséhez.**](#deploy-the-solution)

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] , amely tartalmazza az architektúra diagramja.*

## <a name="architecture"></a>Architektúra

Egy Azure virtuális gép kiépítése további összetevők, például a számítási, hálózati és tárolási erőforrásokat igényel.

* **Erőforráscsoport.** A [erőforráscsoport] [ resource-manager-overview] olyan tároló, amely a kapcsolódó erőforrásokat tárol. Általában érdemes csoportosíthatók az erőforrások egy megoldást élettartamuk és az erőforrások fogják kezelni. Egyetlen virtuális gép számítási feladat érdemes létrehozni az összes erőforrás egyetlen erőforráscsoportként működnek.
* **VM**. Megadhat egy virtuális Gépet, a közzétett lemezképek listájából, vagy egy egyéni felügyelt lemezképről, vagy az Azure-blobtárolóba feltöltött virtuális merevlemez (VHD) fájl.
* **Operációsrendszer-lemez.** Az operációsrendszer-lemez a tárolt virtuális merevlemez [Azure Storage][azure-storage], így azt tartósan fennáll, még ha a gazdagépet nem működik.
* **Ideiglenes lemez.** A virtuális gép létrejön egy ideiglenes lemezt (a `D:` Windows meghajtóján). A lemez a gazdagép egyik fizikai meghajtóján található. Az **nem** mentése az Azure Storage és újraindítások és más virtuális gép életciklus-események során előfordulhat, hogy törölve. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol.
* **Adatlemezek.** Az [adatlemez][data-disk] egy állandó, az alkalmazásadatokhoz használt VHD. Az adatlemezeket (pl. az operációsrendszer-lemezt) az Azure Storage tárolja.
* **Virtuális hálózat (VNet) és alhálózat.** Minden Azure virtuális Gépen központilag telepítik egy Vnetet, amely több szegmentált is lehet.
* **Nyilvános IP-cím.** Egy nyilvános IP-cím van szükség a virtuális gép kommunikálni &mdash; például a távoli asztal (RDP) használatával.  
* **Az Azure DNS**. [Az Azure DNS] [ azure-dns] üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.  
* **Hálózati adapter (NIC)**. Egy olyan hozzárendelt hálózati adapter lehetővé teszi, hogy a virtuális gép és a virtuális hálózat kommunikációját.  
* **Hálózati biztonsági csoport (NSG)**. [Hálózati biztonsági csoportok] [ nsg] használt hálózati erőforráshoz hálózati adatforgalom engedélyezéséhez vagy letiltásához. Egy NSG-t társíthat egy különálló hálózati adapterhez vagy egy alhálózathoz. Ha egy alhálózathoz társítja, az NSG-szabályok az alhálózat összes virtuális gépére vonatkoznak.
* **Diagnosztika.** A diagnosztikai naplózás létfontosságú a virtuális gép kezeléséhez és hibaelhárításához.

## <a name="recommendations"></a>Ajánlatok

Ez az architektúra a baseline javaslatok a Windows virtuális gépek Azure-ban futó jeleníti meg. Azonban nem ajánlott használatával egy virtuális a kritikus fontosságú számítási feladatokhoz, mert azt egy hibaérzékeny pontot hoz létre. A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy [rendelkezésre állási csoportban][availability-set]. További információkért tekintse meg a [több virtuális gép Azure-on való futtatását][multi-vm] ismertető szakaszt. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Az Azure kínál számos különböző virtuális gépek méretét. [Prémium szintű Storage] [ premium-storage] annak magas teljesítménye és kis késleltetése miatt ajánlott, és [adott virtuális gép mérete által támogatott][premium-storage-supported]. Válasszon ki egy ezen méretét a beállítást, hacsak nem rendelkezik a speciális munkaterhelések, például nagy teljesítményű számítástechnikai rendszerek. További információkért lásd: [virtuálisgép-méretek][virtual-machine-sizes].

Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Majd a mértéket a munkaterhelések tényleges Processzor, memória és lemez teljesítménye bemeneti/kimeneti műveletek száma másodpercenként (IOPS), és szükség szerint módosítsa a mérete. Ha a virtuális gép több hálózati adapter van szüksége, vegye figyelembe, hogy a hálózati adapterek maximális száma az egyes van definiálva [Virtuálisgép-méretet][vm-size-tables].

Ha Azure-erőforrások, meg kell adnia egy régiót. A legjobb megoldás, ha a belső felhasználóihoz vagy ügyfeleihez legközelebb eső régiót választja. Azonban nem minden Virtuálisgép-méretek érhetők el minden régióban. További információkért lásd: [régiói][services-by-region]. A Virtuálisgép-méretek érhető el egy adott régióban listáját a következő parancsot a az Azure parancssori felület (CLI):

```
az vm list-sizes --location <location>
```

További információ a közzétett Virtuálisgép-lemezkép kiválasztása: [található Windows Virtuálisgép-rendszerképek][select-vm-image].

Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. Rendszerindítási diagnosztika segíthetnek diagnosztizálni rendszerindítási hiba, ha a virtuális gép nem indítható állapotának beolvasása. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.  

### <a name="disk-and-storage-recommendations"></a>A lemezre és a tárolásra vonatkozó javaslatok

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. Költség a kiépített lemez kapacitása alapul. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség). 

Azt is javasoljuk a használatát [által kezelt lemezeken](/azure/storage/storage-managed-disks-overview). Kezelt lemezeken nincs szükség a storage-fiók. Egyszerűen adja meg, méretének és típusú lemez, és azt egy magas rendelkezésre állású erőforrás van telepítve.

Ha nem felügyelt lemezt használ, hozzon létre külön az Azure storage-fiókok az egyes virtuális gépek a virtuális merevlemezek (VHD), ahhoz, hogy elérte-e elkerülése érdekében a [(IOPS) vonatkozó korlátok] [ vm-disk-limits] storage-fiókok.

Adjon hozzá egy vagy több adatlemezt. A létrehozott VHD-k nincsenek formázva. Jelentkezzen be a virtuális gépek a formázza a lemezt. Ha nem használja a felügyelt lemezek és adatlemezek nagy számú, a teljes i/o-korlátok a tárfiók figyelembe. További információkért lásd a [virtuálisgép-lemez korlátaival kapcsolatos][vm-disk-limits] szakaszt.

Ha lehetséges, alkalmazásokat telepít a adatlemezt, nem az operációsrendszer-lemezképet. Néhány örökölt alkalmazást módosítania kell a C: meghajtón; összetevőinek telepítése Ebben az esetben is [az operációsrendszer-lemez átméretezése] [ resize-os-disk] PowerShell használatával.

Teljesítmény maximalizálása érdekében hozzon létre egy külön tárfiókot diagnosztikai naplók tárolásához. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz.

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

* Tartalék egy [statikus IP-cím] [ static-ip] szüksége van-e a rögzített IP-címet, amely nem változik &mdash; például, ha "A" DNS-rekord létrehozása vagy az IP-cím a biztonságos listához adni.
* Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információkért lásd: [hozzon létre egy teljesen minősített tartománynevet az Azure portálon][fqdn].

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash;

RDP engedélyezéséhez vegyen fel egy NSG-t, amely lehetővé teszi a bejövő forgalom 3389-es TCP-port.

## <a name="scalability-considerations"></a>Méretezési szempontok

Méretezheti a virtuális gépek felfelé vagy lefelé által [módosítása a Virtuálisgép-méretet][vm-resize]. Horizontális vízszintesen, két vagy több virtuális gép egy terheléselosztó mögött elhelyezni. További információkért lásd: [több virtuális gépek Azure-on futó, a méretezhetőség és a rendelkezésre állási][multi-vm].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Magas rendelkezésre állás érdekében telepítsen több virtuális gép rendelkezésre állási csoportba. Ez is biztosít egy magasabb [szolgáltatásiszint-szerződés (SLA) szolgáltatás][vm-sla].

A virtuális gépre hatással lehet egy [tervezett karbantartás][planned-maintenance] vagy egy [nem tervezett karbantartás][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs].

Virtuális merevlemezek vannak tárolva [az Azure storage][azure-storage]. Az Azure storage a tartósság és a rendelkezésre állási replikálja a rendszer.

A normál műveletek során történő véletlen (például hiba miatti) adatvesztés elleni védelem érdekében érdemes időponthoz kötött biztonsági mentéseket is megvalósítani [blob-pillanatképekkel][blob-snapshot] vagy más eszközzel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

**Erőforráscsoportok.** Szorosan kapcsolódó erőforrások, amelyek azonos életciklussal ugyanaz a PUT [erőforráscsoport][resource-manager-overview]. Erőforráscsoportok lehetővé teszik, üzembe helyezését és megfigyelje az erőforrásokat csoportként és számlázási költségek nyomon az erőforráscsoportot. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében. Fontos erőforrásnál nevek egyszerűbbé teheti az adott erőforrás megkeresése, és a szerepkör hozzárendelése. További információkért lásd: [ajánlott vonatkozó elnevezési egyezmény az Azure-erőforrások][naming-conventions].

**Virtuális gépek leállítása.** Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem.

Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha leállítja az operációs rendszer bejelentkezett keresztül, a virtuális gép le van-e állítva, de **nem** felszabadítása. lehetséges, így továbbra is fizetnie kell.

**Virtuális gépek törlése.** Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage].

Véletlen törlés megakadályozhatja, hogy egy [erőforrás zárolási] [ resource-lock] zárolását, a teljes erőforrás csoport vagy zárolási egyéni erőforrások, például egy virtuális Gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használjon [az Azure Security Center] [ security-center] való központi láthatja az Azure-erőforrások biztonsági állapotát. A Security Center a potenciális biztonsági problémákat figyeli, és biztonsági állapotát a központi telepítés átfogó képet nyújt. A Security Center / Azure-előfizetés úgy van beállítva. Biztonsági adatok gyűjtésének leírtak szerint engedélyezze a [az Azure Security Center használatába][security-center-get-started]. Adatgyűjtés engedélyezésekor a rendszer a Security Center automatikusan ellenőrzi a bármely adott előfizetésen belül létrehozott virtuális gépek.

**Javítás kezelése.** Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak. Használjon [csoportházirend-beállítások] [ group-policy] automatikus rendszer-frissítések engedélyezéséhez a virtuális gépen.

**Kártevőirtó.** Ha engedélyezve van, a Security Center ellenőrzi, hogy telepítve van-e a kártevőirtó szoftver. A Security Center az Azure-portálon belül a kártevőirtó szoftver telepítéséhez is használható.

**Műveletek.** Használjon [szerepköralapú hozzáférés-vezérlést (RBAC)] [ rbac] üzembe helyezett Azure-erőforrások elérésének szabályozására. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Bizonyos szerepkörök kifejezetten egy adott Azure-erőforrástípusra jellemzők. Például a virtuális gép közreműködő szerepkört is indítsa újra a virtuális gép felszabadítása, a rendszergazdai jelszó visszaállítása, hozzon létre egy új virtuális Gépet, illetve stb. Más [beépített RBAC-szerepkörök] [ rbac-roles] hasznosak lehetnek a következők ebbe az architektúrába [DevTest Labs felhasználói] [ rbac-devtest] és [ A közreműködői hálózati][rbac-network]. Egy felhasználóhoz több szerepkört is rendelhető, és létrehozhat egyéni szerepköröket a még részletesebb engedélyek érdekében.

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.   

A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás.** Ha titkosítania kell az operációs rendszert és az adatlemezeket, érdemes megfontolnia az [Azure Disk Encryption][disk-encryption] használatát. 

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez az architektúra telepítésének érhető el a [GitHub][github-folder]. Telepíti a következő:

  * Egy virtuális hálózatot egyetlen alhálózattal nevű **webes** üzemeltetni a virtuális Gépet.
  * Egy NSG két bejövő szabályait a virtuális gép RDP és a HTTP-forgalmat engedélyezi.
  * A virtuális gép Windows Server 2016 Datacenter Edition legfrissebb verzióját futtassa.
  * Egy minta egyéni parancsprogramok futtatására szolgáló bővítmény, amely a két adatlemezek formázza, és a PowerShell DSC-parancsfájlt, amely telepíti az IIS.

### <a name="prerequisites"></a>Előfeltételek

Mielőtt a saját előfizetésének telepítése a referencia-architektúrában, a következő lépésekkel kell azt.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [AzureCAT referencia architektúra] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve a számítógépre. A parancssori felület telepítési utasításokért lásd: [Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse a [Azure építőelemeket] [ azbb] npm csomag.

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával használja az alábbi parancsok egyikét, és kövesse az utasításokat.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Az üzembe helyezéséhez azbb használatával

A minta egyetlen virtuális gép számítási feladat telepítéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `virtual-machines\single-vm\parameters\windows` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `single-vm-v2.json` fájlt, és adjon meg egy felhasználónév és SSH-kulcs az idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Futtatás `azbb` a minta virtuális gép telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

A minta referenciaarchitektúra telepítésével kapcsolatos további információkért látogasson el a [GitHub-tárházban][git].

## <a name="next-steps"></a>További lépések

- További tudnivalók a [Azure építőelemeket][azbbv2].
- Telepítése [több virtuális gép] [ multi-vm] az Azure-ban.

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://technet.microsoft.com/en-us/library/dn595129.aspx
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/virtual-machines/windows/premium-storage
[premium-storage-supported]: /azure/virtual-machines/windows/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Az Azure-ban egy Windows virtuális gép architektúrája"
