---
title: "Linux virtuális gép futtatása az Azure-ban"
description: "A Linux virtuális gépek futtatása az Azure-ban a skálázhatóságot, a rugalmasságot, a kezelhetőséget és a biztonságot is figyelembe véve."
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: f0fa86f3f67359cf31543e9dd48b28565b53fbe3
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/17/2018
---
# <a name="run-a-linux-vm-on-azure"></a>Linux virtuális gép futtatása az Azure-ban

Ez a referenciaarchitektúra a Linux virtuális gépek (VM) az Azure-ban történő futtatásához kapcsolódó bevált eljárásokat mutatja be. Javaslatokat tartalmaz a VM, valamint a hálózatkezelési és tárolási összetevők üzembe helyezéséhez. Ez az architektúra egyetlen Virtuálisgép-példány futtatására használható, és összetettebb architektúrák, például N szintű alkalmazások alapjául szolgál. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra ábráját tartalmazó [Visio-fájlt][visio-download].*

## <a name="architecture"></a>Architektúra

Egy Azure-beli virtuális gép üzembe helyezése további összetevőket igényel, például számítási, hálózati és tárolási erőforrásokat.

* **Erőforráscsoport.** Az [erőforráscsoport][resource-manager-overview] egy tároló, amely kapcsolódó erőforrásokat tárol. Általában érdemes az erőforrásokat az élettartamuk alapján csoportosítani, valamint az alapján, hogy ki fogja az erőforrásokat kezelni. Egyetlen virtuális gépet alkalmazó számítási feladat esetén érdemes egyetlen erőforráscsoportot létrehozni az összes erőforráshoz.
* **VM**. A virtuális gépet üzembe helyezheti a közzétett rendszerképek listájáról, egy egyénileg kezelt rendszerképről, vagy egy, az Azure Blob Storage-ba feltöltött virtuálismerevlemez-fájlból (VHD). Az Azure támogatja különböző népszerű Linux-disztribúciók, például a CentOS, a Debian, a Red Hat Enterprise, az Ubuntu és a FreeBSD futtatását. További információk: [Az Azure és a Linux][azure-linux].
* **Operációsrendszer-lemez.** Az operációsrendszer-lemez egy, az [Azure Storage-ban][azure-storage] tárolt VHD, ezért akkor is működik, amikor a gazdagép nem fut. Linux virtuális gépeknél az operációsrendszer-lemez a `/dev/sda1`.
* **Ideiglenes lemez.** A VM egy ideiglenes lemezzel jön létre. A lemez a gazdagép egyik fizikai meghajtóján található. **Nincs** mentve az Azure Storage-ban, és előfordulhat, hogy törlődik az újraindítások és más VM-életciklusesemények során. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol. Linux virtuális gépeknél az ideiglenes lemez a `/dev/sdb1`, és a csatlakoztatásának helye `/mnt/resource` vagy `/mnt`.
* **Adatlemezek.** Az [adatlemez][data-disk] egy állandó, az alkalmazásadatokhoz használt VHD. Az adatlemezeket (pl. az operációsrendszer-lemezt) az Azure Storage tárolja.
* **Virtuális hálózat (VNet) és alhálózat.** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve.
* **Nyilvános IP-cím.** Egy nyilvános IP-címre van szükség ahhoz, hogy a virtuális géppel kommunikálni lehessen – például SSH-n keresztül.
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Hálózati adapter (NIC)**. Egy hozzárendelt hálózati adapter teszi lehetővé a virtuális gép számára a virtuális hálózattal való kommunikációt.
* **Hálózati biztonsági csoport (NSG)**. A [Hálózati biztonsági csoportok][nsg] a hálózati erőforrás felé irányuló hálózati adatforgalom engedélyezéséhez vagy letiltásához használhatók. Egy NSG-t társíthat egy különálló hálózati adapterhez vagy egy alhálózathoz. Ha egy alhálózathoz társítja, az NSG-szabályok az alhálózat összes virtuális gépére vonatkoznak.
* **Diagnosztika.** A diagnosztikai naplózás létfontosságú a virtuális gép kezeléséhez és hibaelhárításához.

## <a name="recommendations"></a>Javaslatok

Ez az architektúra a Linux virtuális gépek az Azure-ban való futtatásának általános javaslatait mutatja be. Nem javasoljuk azonban, hogy egyetlen virtuális gépet használjon kritikus fontosságú számítási feladatokhoz, mert azzal kritikus meghibásodási pontot hoz létre. A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy [rendelkezésre állási csoportban][availability-set]. További információkért tekintse meg a [több virtuális gép Azure-on való futtatását][multi-vm] ismertető szakaszt. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Az Azure számos különböző méretét kínál a virtuális gépekhez. A [Premium Storage][premium-storage] magas teljesítménye és kis késleltetése miatt ajánlott, és [bizonyos virtuálisgép-méretek támogatják][premium-storage-supported]. Válassza ezen méretek egyikét, kivéve, ha speciális számítási feladatokhoz, például nagy teljesítményű feldolgozáshoz kívánja a gépeket használni. További információt a [virtuálisgép-méretekről szóló részben][virtual-machine-sizes] talál.

Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Ezután mérje meg a valós számítási feladat teljesítményét a CPU, a memória és a lemez másodpercenkénti bemeneti/kimeneti műveletei (IOPS) figyelembe vételével, és módosítsa a méretet, ha szükséges. Ha több hálózati adapterre van szükség a virtuális géphez, vegye figyelembe, hogy a hálózati adapterek maximális száma minden [virtuálisgép-mérethez][vm-size-tables] külön van meghatározva.

Amikor üzembe helyezi az Azure-erőforrásokat, meg kell adnia egy régiót. A legjobb megoldás, ha a belső felhasználóihoz vagy ügyfeleihez legközelebb eső régiót választja. Azonban nem minden virtuálisgép-méret érhető el minden régióban. További információ: [Szolgáltatások régiónként][services-by-region]. Az egy adott régióban elérhető virtuálisgép-méretek listájának megtekintéséhez futtassa a következő parancsot az Azure parancssori felületéről (CLI):

```
az vm list-sizes --location <location>
```

A közzétett Virtuálisgép-lemezképek kiválasztásával kapcsolatban további információt a [Linux virtuálisgép-rendszerképek megkeresésével][select-vm-image] foglalkozó részben talál.

Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. A rendszerindítási diagnosztika segít diagnosztizálni a rendszerindítási hibát, ha a virtuális gép nem indítható állapotba kerül. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.  

### <a name="disk-and-storage-recommendations"></a>A lemezre és a tárolásra vonatkozó javaslatok

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. A költség az üzembe helyezett lemez kapacitásától függően változik. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség). 

Azt is javasoljuk, hogy [felügyelt lemezeket](/azure/storage/storage-managed-disks-overview) használjon. A felügyelt lemezek nem igényelnek tárfiókot. Egyszerűen adja meg a méretet és a lemez típusát, és az magas rendelkezésre állású erőforrásként lesz üzembe helyezve.

Ha nem felügyelt lemezeket használ, hozzon létre külön Azure Storage-fiókot minden virtuális géphez a virtuális merevlemezek (VHD-k) tárolására, hogy elkerülje az [IOPS-korlátok][vm-disk-limits] elérését a tárfiókban.

Adjon hozzá egy vagy több adatlemezt. A létrehozott VHD-k nincsenek formázva. A lemez formázásához jelentkezzen be a virtuális gépre. Ha nem felügyelt lemezeket használ, és nagy számú adatlemezzel rendelkezik, vegye figyelembe a tárfiók teljes I/O-korlátját. További információkért lásd a [virtuálisgép-lemez korlátaival kapcsolatos][vm-disk-limits] szakaszt.

A Linux felületen az adatlemezek `/dev/sdc`, `/dev/sdd` stb. formátumban jelennek meg. Az `lsblk` futtatásával listázhatja a blokkeszközöket, beleértve a lemezeket. Adatlemez használatához hozzon létre egy partíciót és egy fájlrendszert, majd csatlakoztassa a lemezt. Példa:

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

Adatlemez hozzáadásakor a rendszer hozzárendel egy logikaiegység-szám (LUN) azonosítót a lemezhez. Lehetősége van megadni a LUN azonosítót &mdash; – ha például lecserél egy lemezt, de meg akarja tartani a LUN azonosítót, vagy ha egy alkalmazása egy konkrét LUN azonosítót keres. Ne feledje azonban, hogy az egyes lemezek LUN azonosítóinak egyedinek kell lenniük.

Érdemes lehet módosítani az I/O-ütemezőt az SSD-k teljesítményének optimalizálására, mert a prémium szintű tárfiókkal rendelkező virtuális gépek lemezei SSD-k. Általában az NOOP-ütemezőt ajánlott használni SSD-khez, de érdemes az [iostat] vagy hasonló eszközt használni az adott számítási feladathoz tartozó lemez I/O-teljesítményének monitorozására.

A legjobb teljesítmény érdekében hozzon létre egy önálló tárfiókot a diagnosztikai naplók tárolására. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz.

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

* Akkor foglaljon le [statikus IP-címet][static-ip], ha rögzített, nem módosuló IP-címre van szüksége – például ha egy A rekordot kell létrehoznia a DNS-ben, vagy ha hozzá kell adnia az IP-címet a biztonságos elemek listájához.&mdash;
* Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információért tekintse meg a [Teljes tartománynév létrehozása az Azure Portalon][fqdn] szakaszt. Használhatja az [Azure DNS-t][azure-dns] vagy más DNS-szolgáltatást.

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash;

Az SSH engedélyezéséhez adjon hozzá egy NSG-szabályt, amely engedélyezi a bejövő forgalmat a 22-es TCP-porton.

## <a name="scalability-considerations"></a>Méretezési szempontok

A virtuális gépeket a [virtuális gép méretének módosításával][vm-resize] skálázhatja. A horizontális felskálázáshoz helyezzen két vagy több virtuális gépet egy terheléselosztó mögé. További információért tekintse meg a [Több virtuális gép Azure-on való futtatása a skálázhatóság és a rendelkezésre állás érdekében][multi-vm] témakörrel foglalkozó szakaszt.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy rendelkezésre állási csoportban. Ez magasabb szintű [szolgáltatói szerződést (SLA-t)][vm-sla] biztosít.

A virtuális gépre hatással lehet egy [tervezett karbantartás][planned-maintenance] vagy egy [nem tervezett karbantartás][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs].

Az adatlemezeket az [Azure Storage][azure-storage] tárolja. Az Azure Storage a tartósság és a rendelkezésre állás érdekében replikálva van.

A normál műveletek során történő véletlen (például hiba miatti) adatvesztés elleni védelem érdekében érdemes időponthoz kötött biztonsági mentéseket is megvalósítani [blob-pillanatképekkel][blob-snapshot] vagy más eszközzel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

**Erőforráscsoportok.** A szorosan összekapcsolt, azonos életciklusú erőforrásokat helyezze egy [erőforráscsoportba][resource-manager-overview]. Az erőforráscsoportok segítségével csoportosan helyezhet üzembe és monitorozhat erőforrásokat, és a számlázási költségeket erőforráscsoportonként követheti. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében. Jelentéssel bíró erőforrásneveket adjon meg, hogy egyszerűbben megkereshesse és azonosíthassa az egyes erőforrásokat és azok szerepkörét. További információ: [Az Azure-erőforrások ajánlott elnevezési konvenciói][naming-conventions].

**SSH**. Linux virtuális gép létrehozása előtt hozzon létre egy 2048 bites RSA nyilvános-titkos kulcspárt. A virtuális gép létrehozásakor használja a nyilvánoskulcs-fájlt. További információ: [SSH használata Linuxon és Macen az Azure-on][ssh-linux].

**Virtuális gépek leállítása.** Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem.

Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha az operációs rendszerből állítja le, amikor be van jelentkezve, azzal a virtuális gépet leállítja, de **nem** szabadítja fel, tehát továbbra is fizetnie kell a díját.

**Virtuális gépek törlése.** Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage].

A véletlen törlés megelőzése érdekében használjon [erőforrászárat][resource-lock]. Ezzel zárolhat egy egész erőforráscsoportot, vagy egyes erőforrásokat, például egy virtuális gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használja az [Azure Security Centert][security-center] az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez. A Security Center monitorozza a potenciális biztonsági problémákat, és átfogó képet nyújt az üzemi környezet biztonsági állapotáról. A Security Center Azure-előfizetésenként külön konfigurálandó. Engedélyezze a biztonsági adatok gyűjtését [az Azure Security Center rövid útmutatójában leírtak szerint][security-center-get-started]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

**Javítások kezelése.** Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak-e. 

**Kártevőirtó.** Ha engedélyezve van, a Security Center ellenőrzi, hogy van-e telepítve kártevőirtó szoftver. A Security Center segítségével telepíthet is kártevőirtó szoftvert az Azure Portalon belülről.

**Műveletek.** A [szerepköralapú hozzáférés-vezérléssel (RBAC)][rbac] szabályozható az üzembe helyezett Azure-erőforrásokhoz való hozzáférés. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Bizonyos szerepkörök kifejezetten egy adott Azure-erőforrástípusra jellemzők. A Virtuális gépek közreműködője szerepkör például újraindíthat vagy felszabadíthat egy virtuális gépet, alaphelyzetbe állíthatja a rendszergazdai jelszót, létrehozhat egy új virtuális gépet, stb. Ehhez az architektúrához hasznos lehet még a [DevTest Labs-felhasználó][rbac-devtest] és a [Hálózati közreműködő][rbac-network] [beépített RBAC-szerepkör][rbac-roles]. Egy felhasználóhoz több szerepkört is rendelhető, és létrehozhat egyéni szerepköröket a még részletesebb engedélyek érdekében.

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.   

A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás.** Ha titkosítania kell az operációs rendszert és az adatlemezeket, érdemes megfontolnia az [Azure Disk Encryption][disk-encryption] használatát. 

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. A következőket helyezi üzembe:

  * Egy virtuális hálózat egyetlen alhálózattal (neve **web**), amely a virtuális gépet üzemelteti.
  * Egy két bejövő szabállyal rendelkező NSG, amely engedélyezi az SSH- és a HTTP-forgalmat a virtuális gép felé.
  * Egy virtuális gép, amely az Ubuntu 16.04.3 LTS legújabb verzióját futtatja.
  * Egy egyéni minta szkriptbővítmény, amely formázza a két adatlemezt, és üzembe helyezi az Apache HTTP Servert az Ubuntu virtuális gépen.

### <a name="prerequisites"></a>Előfeltételek

Mielőtt üzembe helyezhetné saját előfizetésében a referenciaarchitektúrát, az alábbi lépéseket kell elvégeznie.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén. Információk a CLI telepítésével kapcsolatban: [Az Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

4. Jelentkezzen be Azure-fiókjába egy parancssorból, Bash-parancssorból vagy PowerShell-parancssorból az alábbi parancsok egyikével, és kövesse az utasításokat.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>A megoldás üzembe helyezése az azbb használatával

A mintául szolgáló, egyetlen virtuális gépet alkalmazó számítási feladat üzembe helyezéséhez kövesse az alábbi lépéseket:

1. Navigáljon az előző lépésekben letöltött adattár `virtual-machines\single-vm\parameters\linux` mappájához.

2. Nyissa meg a `single-vm-v2.json` fájlt, és adjon meg egy felhasználónevet és egy nyilvános SSH-kulcsot az idézőjelek között az alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. Futtassa az `azbb` parancsot az alábbi módon a minta virtuális gép telepítéséhez.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

A mintául szolgáló referenciaarchitektúra üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.

## <a name="next-steps"></a>További lépések

- További tudnivalók az [Azure építőelemeiről][azbbv2].
- [Több virtuális gép][multi-vm] telepítése az Azure-ban.

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Önálló Linux virtuális gép architektúra az Azure-ban"
