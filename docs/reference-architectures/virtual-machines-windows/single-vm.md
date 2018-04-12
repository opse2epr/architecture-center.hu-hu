---
title: Windows virtuális gép futtatása az Azure-ban
description: A Windows virtuális gépek futtatása az Azure-ban a skálázhatóságot, a rugalmasságot, a kezelhetőséget és a biztonságot is figyelembe véve.
author: telmosampaio
ms.date: 04/03/2018
pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 9bc0b4af56b9194fd1bec8a189c86963ad2b0c98
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="run-a-windows-vm-on-azure"></a>Windows virtuális gép futtatása az Azure-ban

Ez a referenciaarchitektúra a Windows virtuális gépek (VM) az Azure-ban történő futtatásához kapcsolódó bevált eljárásokat mutatja be. Javaslatokat tartalmaz a VM, valamint a hálózatkezelési és tárolási összetevők üzembe helyezéséhez. Ez az architektúra egyetlen Virtuálisgép-példány futtatására használható, és összetettebb architektúrák, például N szintű alkalmazások alapjául szolgál. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra ábráját tartalmazó [Visio-fájlt][visio-download].*

## <a name="architecture"></a>Architektúra

Egy Azure virtuális gép kiépítése néhány további összetevők mellett a virtuális gépért, beleértve a hálózati és tárolási erőforrásokat igényel.

* **Erőforráscsoport.** A [erőforráscsoport] [ resource-manager-overview] , amely tárolja a kapcsolódó Azure-erőforrások logikai tárolója. Általában csoportosíthatja az erőforrásokat az élettartamuk alapján, és fogják kezelni azokat. 

* **VM**. A virtuális gépet üzembe helyezheti a közzétett rendszerképek listájáról, egy egyénileg kezelt rendszerképről, vagy egy, az Azure Blob Storage-ba feltöltött virtuálismerevlemez-fájlból (VHD).

* **Által kezelt lemezeken**. [Azure-lemezeket felügyelt] [ managed-disks] a tárterület kezelése, a Lemezkezelés egyszerűbbé. Az operációsrendszer-lemez egy, az [Azure Storage-ban][azure-storage] tárolt VHD, ezért akkor is működik, amikor a gazdagép nem fut. Javasoljuk továbbá egy vagy több létrehozása [adatlemezek][data-disk], amelyeket az alkalmazásadatok használt állandó virtuális merevlemezek.

* **Ideiglenes lemez.** A virtuális gép egy ideiglenes lemezzel jön létre (a `D:` meghajtó a Windowsban). A lemez a gazdagép egyik fizikai meghajtóján található. *Nincs* mentve az Azure Storage-ban, és előfordulhat, hogy törlődik az újraindítások és más VM-életciklusesemények során. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol.

* **Virtuális hálózathoz (VNet).** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve.

* **Hálózati adapter (NIC)**. A hálózati adapter teszi lehetővé a virtuális gép számára a virtuális hálózattal való kommunikációt.  

* **Nyilvános IP-cím.** Egy nyilvános IP-címre van szükség ahhoz, hogy a virtuális géppel kommunikálni lehessen – például távoli asztali kapcsolaton (RDP) keresztül.  

* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

* **Hálózati biztonsági csoport (NSG)**. [Hálózati biztonsági csoportok] [ nsg] engedélyezi vagy megtagadja a hálózati forgalmat a virtuális gépekhez használt. Az NSG-k társítani, alhálózatok vagy Virtuálisgép-példányokhoz lehet. 

* **Diagnosztika.** A diagnosztikai naplózás létfontosságú a virtuális gép kezeléséhez és hibaelhárításához.

## <a name="recommendations"></a>Javaslatok

Ez az architektúra a Windows virtuális gépek az Azure-ban való futtatásának általános javaslatait mutatja be. Nem javasoljuk azonban, hogy egyetlen virtuális gépet használjon kritikus fontosságú számítási feladatokhoz, mert azzal kritikus meghibásodási pontot hoz létre. A magas rendelkezésre állás érdekében két vagy több terheléselosztással rendelkező virtuális gépek telepítéséhez. További információkért tekintse meg a [több virtuális gép Azure-on való futtatását][multi-vm] ismertető szakaszt.

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Az Azure számos különböző méretét kínál a virtuális gépekhez. További információkért lásd: [az Azure virtuális gépek méretei][virtual-machine-sizes]. Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Ezután mérje meg a valós számítási feladat teljesítményét a CPU, a memória és a lemez másodpercenkénti bemeneti/kimeneti műveletei (IOPS) figyelembe vételével, és módosítsa a méretet, ha szükséges. Ha több hálózati adapterre van szükség a virtuális géphez, vegye figyelembe, hogy a hálózati adapterek maximális száma minden [virtuálisgép-mérethez][vm-size-tables] külön van meghatározva.

Általában válassza ki a belső felhasználók vagy az ügyfelek legközelebbi Azure-régióban. Azonban nem minden virtuálisgép-méret érhető el minden régióban. További információ: [Szolgáltatások régiónként][services-by-region]. Az egy adott régióban elérhető virtuálisgép-méretek listájának megtekintéséhez futtassa a következő parancsot az Azure parancssori felületéről (CLI):

```
az vm list-sizes --location <location>
```

A közzétett Virtuálisgép-lemezképek kiválasztásával kapcsolatban további információt a [Windows virtuálisgép-rendszerképek megkeresésével][select-vm-image] foglalkozó részben talál.

Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. A rendszerindítási diagnosztika segít diagnosztizálni a rendszerindítási hibát, ha a virtuális gép nem indítható állapotba kerül. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.  

### <a name="disk-and-storage-recommendations"></a>A lemezre és a tárolásra vonatkozó javaslatok

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. A költség az üzembe helyezett lemez kapacitásától függően változik. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség). 

Javasoljuk továbbá használatával [kezelt lemezek][managed-disks]. A felügyelt lemezek nem igényelnek tárfiókot. Egyszerűen adja meg a méretet és a lemez típusát, és az magas rendelkezésre állású erőforrásként lesz üzembe helyezve.

Adjon hozzá egy vagy több adatlemezt. A létrehozott VHD-k nincsenek formázva. A lemez formázásához jelentkezzen be a virtuális gépre. Ha lehetséges, az alkalmazásokat adatlemezre telepítse, ne az operációs rendszer lemezére. Előfordulhat, hogy néhány örökölt alkalmazásnak összetevőket kell telepítenie a C: meghajtóra. Ebben az esetben [az operációsrendszer-lemezt átméretezheti][resize-os-disk] a PowerShell használatával.

Hozzon létre egy tárfiókot, diagnosztikai naplók tárolásához. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz.

> [!NOTE]
> Ha nem kezelt lemezek, hozzon létre külön az Azure storage-fiókok az egyes virtuális gépek a virtuális merevlemezek (VHD), ahhoz, hogy elérte-e elkerülése érdekében a [(IOPS) vonatkozó korlátok] [ vm-disk-limits] storage-fiókok. Vegye figyelembe a teljes i/o-korlátozás a tárfiók. További információkért lásd a [virtuálisgép-lemez korlátaival kapcsolatos][vm-disk-limits] szakaszt.


### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

* Akkor foglaljon le [statikus IP-címet][static-ip], ha rögzített, nem módosuló IP-címre van szüksége – például ha egy „A” DNS-rekordot kell létrehoznia, vagy ha hozzá kell adnia az IP-címet a biztonságos elemek listájához.
* Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információért tekintse meg a [Teljes tartománynév létrehozása az Azure Portalon][fqdn] szakaszt.

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash;

Az RDP engedélyezéséhez adjon hozzá egy NSG-szabályt, amely engedélyezi a bejövő forgalmat a 3389-es TCP-porton.

## <a name="scalability-considerations"></a>Méretezési szempontok

A virtuális gépeket a [virtuális gép méretének módosításával][vm-resize] skálázhatja. A horizontális felskálázáshoz helyezzen két vagy több virtuális gépet egy terheléselosztó mögé. További információkért lásd: [futtassa a virtuális gépek elosztott terhelésű a méretezhetőség és a rendelkezésre állási][multi-vm].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy rendelkezésre állási csoportban. Ez magasabb szintű [szolgáltatói szerződést (SLA-t)][vm-sla] biztosít.

A virtuális gépre hatással lehet egy [tervezett karbantartás][planned-maintenance] vagy egy [nem tervezett karbantartás][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs].

A normál műveletek során történő véletlen (például hiba miatti) adatvesztés elleni védelem érdekében érdemes időponthoz kötött biztonsági mentéseket is megvalósítani [blob-pillanatképekkel][blob-snapshot] vagy más eszközzel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

**Erőforráscsoportok.** A szorosan összekapcsolt, azonos életciklusú erőforrásokat helyezze egy [erőforráscsoportba][resource-manager-overview]. Az erőforráscsoportok segítségével csoportosan helyezhet üzembe és monitorozhat erőforrásokat, és a számlázási költségeket erőforráscsoportonként követheti. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében. Jelentéssel bíró erőforrásneveket adjon meg, hogy egyszerűbben megkereshesse és azonosíthassa az egyes erőforrásokat és azok szerepkörét. További információ: [Az Azure-erőforrások ajánlott elnevezési konvenciói][naming-conventions].

**Virtuális gépek leállítása.** Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem. Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha az operációs rendszerből állítja le, amikor be van jelentkezve, azzal a virtuális gépet leállítja, de **nem** szabadítja fel, tehát továbbra is fizetnie kell a díját.

**Virtuális gépek törlése.** Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage]. A véletlen törlés megelőzése érdekében használjon [erőforrászárat][resource-lock]. Ezzel zárolhat egy egész erőforráscsoportot, vagy egyes erőforrásokat, például egy virtuális gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használja az [Azure Security Centert][security-center] az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez. A Security Center monitorozza a potenciális biztonsági problémákat, és átfogó képet nyújt az üzemi környezet biztonsági állapotáról. A Security Center Azure-előfizetésenként külön konfigurálandó. Engedélyezze a biztonsági adatok gyűjtését [az Azure Security Center rövid útmutatójában leírtak szerint][security-center-get-started]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

**Javítások kezelése.** Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak-e. Használjon [Csoportszabályzat-beállításokat][group-policy] a virtuális gépen az automatikus rendszerfrissítések engedélyezéséhez.

**Kártevőirtó.** Ha engedélyezve van, a Security Center ellenőrzi, hogy van-e telepítve kártevőirtó szoftver. A Security Center segítségével telepíthet is kártevőirtó szoftvert az Azure Portalon belülről.

**Műveletek.** A [szerepköralapú hozzáférés-vezérléssel (RBAC)][rbac] szabályozható az üzembe helyezett Azure-erőforrásokhoz való hozzáférés. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Bizonyos szerepkörök kifejezetten egy adott Azure-erőforrástípusra jellemzők. A Virtuális gépek közreműködője szerepkör például újraindíthat vagy felszabadíthat egy virtuális gépet, alaphelyzetbe állíthatja a rendszergazdai jelszót, létrehozhat egy új virtuális gépet, stb. Ehhez az architektúrához hasznos lehet még a [DevTest Labs-felhasználó][rbac-devtest] és a [Hálózati közreműködő][rbac-network] [beépített RBAC-szerepkör][rbac-roles]. Egy felhasználóhoz több szerepkört is rendelhető, és létrehozhat egyéni szerepköröket a még részletesebb engedélyek érdekében.

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.   

A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás.** Ha titkosítania kell az operációs rendszert és az adatlemezeket, érdemes megfontolnia az [Azure Disk Encryption][disk-encryption] használatát. 

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. A következőket helyezi üzembe:

  * Egy virtuális hálózat egyetlen alhálózattal (neve **web**), amely a virtuális gépet üzemelteti.
  * Egy két bejövő szabállyal rendelkező NSG, amely engedélyezi az RDP- és a HTTP-forgalmat a virtuális gép felé.
  * Olyan virtuális gép, amely a Windows Server 2016 Datacenter Edition legfrissebb verzióját futtatja.
  * Egy minta egyéni parancsprogramok futtatására szolgáló bővítmény, amely a két adatlemezek formázza, és a PowerShell DSC-parancsfájlt, amely telepíti az Internet Information Services (IIS).

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén. Információk a CLI telepítésével kapcsolatban: [Az Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

4. Egy parancssori ablakot, bash, vagy PowerShell kérdés adja meg a következő parancs futtatásával jelentkezzen be az Azure-fiókjával.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>A megoldás üzembe helyezése az azbb használatával

A referencia-architektúrában telepítéséhez kövesse az alábbi lépéseket:

1. Navigáljon az előző lépésekben letöltött adattár `virtual-machines\single-vm\parameters\windows` mappájához.

2. Nyissa meg a `single-vm-v2.json` fájlt, és adja meg a felhasználónevet és jelszót az idézőjelek között, majd mentse a fájlt.

   ```bash
   "adminUsername": "",
   "adminPassword": "",
   ```

3. Futtassa az `azbb` parancsot az alábbi módon a minta virtuális gép telepítéséhez.

   ```bash
   azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
   ```

A telepítés ellenőrzése, hogy a következő paranccsal Azure CLI található a virtuális gép a nyilvános IP-cím:

```bash
az vm show -n ra-single-windows-vm1 -g <resource-group-name> -d -o table
```

Ha ezt a címet egy webböngészőben nyissa meg az alapértelmezett IIS-kezdőlap kell megjelennie.

A központi telepítés testreszabásával kapcsolatos további tudnivalókért keresse fel a [GitHub-tárházban][git].

## <a name="next-steps"></a>További lépések

- További tudnivalók az [Azure építőelemeiről][azbbv2].
- [Több virtuális gép][multi-vm] telepítése az Azure-ban.

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
[group-policy]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn595129(v=ws.11)
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[multi-vm]: multi-vm.md
[naming-conventions]: ../../best-practices/naming-conventions.md
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Önálló Windows virtuális gép architektúra az Azure-ban"
