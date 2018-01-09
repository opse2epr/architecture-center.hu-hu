---
title: "Linux virtuális gép futtatása az Azure-on"
description: "Az Azure, a méretezhetőséget, a rugalmasság, a kezelhetőségi és a biztonsági kifizető figyelmet Linux virtuális gép futtatásának módját."
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 7caef46e53b42011b5a12ef53384c0352b9b9a72
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2018
---
# <a name="run-a-linux-vm-on-azure"></a>Linux virtuális gép futtatása az Azure-on

A referencia-architektúrában jeleníti meg az Azure-on futó Linux virtuális gépek (VM) bevált gyakorlatok csoportja. Ez magában foglalja a hálózati és tárolási összetevők mellett a virtuális gép üzembe helyezéséhez javaslatok. Ez az architektúra egyetlen Virtuálisgép-példány futtatása is használható, és alapjául szolgáló összetettebb architektúrák például N szintű alkalmazásokhoz. [**A megoldás üzembe helyezéséhez.**](#deploy-the-solution)

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] , amely tartalmazza az architektúra diagramja.*

## <a name="architecture"></a>Architektúra

Egy Azure virtuális gép kiépítése további összetevők, például a számítási, hálózati és tárolási erőforrásokat igényel.

* **Erőforráscsoport.** A [erőforráscsoport] [ resource-manager-overview] olyan tároló, amely a kapcsolódó erőforrásokat tárol. Általában érdemes csoportosíthatók az erőforrások egy megoldást élettartamuk és az erőforrások fogják kezelni. Egyetlen virtuális gép számítási feladat érdemes létrehozni az összes erőforrás egyetlen erőforráscsoportként működnek.
* **VM**. Megadhat egy virtuális Gépet, a közzétett lemezképek listájából, vagy egy egyéni felügyelt lemezképről, vagy az Azure-blobtárolóba feltöltött virtuális merevlemez (VHD) fájl. Az Azure támogatja különböző népszerű Linux-disztribúciók, például a CentOS, a Debian, a Red Hat Enterprise, az Ubuntu és a FreeBSD futtatását. További információk: [Az Azure és a Linux][azure-linux].
* **Operációsrendszer-lemez.** Az operációsrendszer-lemez a tárolt virtuális merevlemez [Azure Storage][azure-storage], így azt tartósan fennáll, még ha a gazdagépet nem működik. Linux virtuális gépekhez, az operációsrendszer-lemez van `/dev/sda1`.
* **Ideiglenes lemez.** A VM egy ideiglenes lemezzel jön létre. A lemez a gazdagép egyik fizikai meghajtóján található. Az **nem** mentése az Azure Storage és újraindítások és más virtuális gép életciklus-események során előfordulhat, hogy törölve. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol. Linux virtuális gépekhez, ideiglenes lemez `/dev/sdb1` és csatlakoztatva `/mnt/resource` vagy `/mnt`.
* **Adatlemezek.** Az [adatlemez][data-disk] egy állandó, az alkalmazásadatokhoz használt VHD. Az adatlemezeket (pl. az operációsrendszer-lemezt) az Azure Storage tárolja.
* **Virtuális hálózat (VNet) és alhálózat.** Minden Azure virtuális Gépen központilag telepítik egy Vnetet, amely több szegmentált is lehet.
* **Nyilvános IP-cím.** Egy nyilvános IP-cím van szükség a virtuális gép kommunikálni &mdash; például ssh.
* **Az Azure DNS**. [Az Azure DNS] [ azure-dns] üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Hálózati adapter (NIC)**. Egy olyan hozzárendelt hálózati adapter lehetővé teszi, hogy a virtuális gép és a virtuális hálózat kommunikációját.
* **Hálózati biztonsági csoport (NSG)**. [Hálózati biztonsági csoportok] [ nsg] használt hálózati erőforráshoz hálózati adatforgalom engedélyezéséhez vagy letiltásához. Egy NSG-t társíthat egy különálló hálózati adapterhez vagy egy alhálózathoz. Ha egy alhálózathoz társítja, az NSG-szabályok az alhálózat összes virtuális gépére vonatkoznak.
* **Diagnosztika.** A diagnosztikai naplózás létfontosságú a virtuális gép kezeléséhez és hibaelhárításához.

## <a name="recommendations"></a>Javaslatok

Ez az architektúra látható a baseline javaslatok az Azure-ban futó Linux virtuális gép. Azonban nem ajánlott használatával egy virtuális a kritikus fontosságú számítási feladatokhoz, mert azt egy hibaérzékeny pontot hoz létre. A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy [rendelkezésre állási csoportban][availability-set]. További információkért tekintse meg a [több virtuális gép Azure-on való futtatását][multi-vm] ismertető szakaszt. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Az Azure kínál számos különböző virtuális gépek méretét. [Prémium szintű Storage] [ premium-storage] annak magas teljesítménye és kis késleltetése miatt ajánlott, és [adott virtuális gép mérete által támogatott][premium-storage-supported]. Válasszon ki egy ezen méretét a beállítást, hacsak nem rendelkezik a speciális munkaterhelések, például nagy teljesítményű számítástechnikai rendszerek. További információkért lásd: [virtuálisgép-méretek][virtual-machine-sizes].

Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Majd a mértéket a munkaterhelések tényleges Processzor, memória és lemez teljesítménye bemeneti/kimeneti műveletek száma másodpercenként (IOPS), és szükség szerint módosítsa a mérete. Ha a virtuális gép több hálózati adapter van szüksége, vegye figyelembe, hogy a hálózati adapterek maximális száma az egyes van definiálva [Virtuálisgép-méretet][vm-size-tables].

Ha Azure-erőforrások, meg kell adnia egy régiót. A legjobb megoldás, ha a belső felhasználóihoz vagy ügyfeleihez legközelebb eső régiót választja. Azonban nem minden Virtuálisgép-méretek érhetők el minden régióban. További információkért lásd: [régiói][services-by-region]. A Virtuálisgép-méretek érhető el egy adott régióban listáját a következő parancsot a az Azure parancssori felület (CLI):

```
az vm list-sizes --location <location>
```

További információ a közzétett Virtuálisgép-lemezkép kiválasztása: [található Linux Virtuálisgép-rendszerképek][select-vm-image].

Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. Rendszerindítási diagnosztika segíthetnek diagnosztizálni rendszerindítási hiba, ha a virtuális gép nem indítható állapotának beolvasása. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.  

### <a name="disk-and-storage-recommendations"></a>A lemezre és a tárolásra vonatkozó javaslatok

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. Költség a kiépített lemez kapacitása alapul. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség). 

Azt is javasoljuk a használatát [által kezelt lemezeken](/azure/storage/storage-managed-disks-overview). Kezelt lemezeken nincs szükség a storage-fiók. Egyszerűen adja meg, méretének és típusú lemez, és azt egy magas rendelkezésre állású erőforrás van telepítve.

Ha nem felügyelt lemezt használ, hozzon létre külön az Azure storage-fiókok az egyes virtuális gépek a virtuális merevlemezek (VHD), ahhoz, hogy elérte-e elkerülése érdekében a [(IOPS) vonatkozó korlátok] [ vm-disk-limits] storage-fiókok.

Adjon hozzá egy vagy több adatlemezt. A létrehozott VHD-k nincsenek formázva. Jelentkezzen be a virtuális gépek a formázza a lemezt. Ha nem használja a felügyelt lemezek és adatlemezek nagy számú, a teljes i/o-korlátok a tárfiók figyelembe. További információkért lásd a [virtuálisgép-lemez korlátaival kapcsolatos][vm-disk-limits] szakaszt.

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

Érdemes lehet módosítani az i/o-ütemező optimalizálása SSD-k teljesítményt, mert a lemez virtuális gépek a prémium szintű tárfiókok SSD-k. A közös javasoljuk, hogy használja a NOOP Feladatütemező SSD-k, de például ajánlott egy eszköz [iostat] a munkaterheléshez lemez i/o-teljesítményének figyelése.

Teljesítmény maximalizálása érdekében hozzon létre egy külön tárfiókot diagnosztikai naplók tárolásához. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz.

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

* Akkor foglaljon le [statikus IP-címet][static-ip], ha rögzített, nem módosuló IP-címre van szüksége – például ha egy A rekordot kell létrehoznia a DNS-ben, vagy ha hozzá kell adnia az IP-címet a biztonságos elemek listájához.&mdash;
* Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információkért lásd: [hozzon létre egy teljesen minősített tartománynevet az Azure portálon][fqdn]. Használhat [Azure DNS] [ azure-dns] vagy egy másik DNS-szolgáltatás.

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash;

SSH engedélyezéséhez vegyen fel egy NSG-t, amely lehetővé teszi, hogy a 22-es TCP-portot a bejövő forgalmat.

## <a name="scalability-considerations"></a>Méretezési szempontok

Méretezheti a virtuális gépek felfelé vagy lefelé által [módosítása a Virtuálisgép-méretet][vm-resize]. Horizontális vízszintesen, két vagy több virtuális gép egy terheléselosztó mögött elhelyezni. További információkért lásd: [több virtuális gépek Azure-on futó, a méretezhetőség és a rendelkezésre állási][multi-vm].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Magas rendelkezésre állás érdekében telepítsen több virtuális gép rendelkezésre állási csoportba. Ez is biztosít egy magasabb [szolgáltatásiszint-szerződés (SLA) szolgáltatás][vm-sla].

A virtuális gépre hatással lehet egy [tervezett karbantartás][planned-maintenance] vagy egy [nem tervezett karbantartás][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs].

Virtuális merevlemezek vannak tárolva [az Azure storage][azure-storage]. Az Azure storage a tartósság és a rendelkezésre állási replikálja a rendszer.

A normál műveletek során történő véletlen (például hiba miatti) adatvesztés elleni védelem érdekében érdemes időponthoz kötött biztonsági mentéseket is megvalósítani [blob-pillanatképekkel][blob-snapshot] vagy más eszközzel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

**Erőforráscsoportok.** Szorosan kapcsolódó erőforrások, amelyek azonos életciklussal ugyanaz a PUT [erőforráscsoport][resource-manager-overview]. Erőforráscsoportok lehetővé teszik, üzembe helyezését és megfigyelje az erőforrásokat csoportként és számlázási költségek nyomon az erőforráscsoportot. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében. Fontos erőforrásnál nevek egyszerűbbé teheti az adott erőforrás megkeresése, és a szerepkör hozzárendelése. További információkért lásd: [Azure-erőforrások elnevezési szabályai ajánlott][naming-conventions].

**SSH**. Linux virtuális gép létrehozása előtt hozzon létre egy 2048 bites RSA nyilvános-titkos kulcspárt. A virtuális gép létrehozásakor használja a nyilvánoskulcs-fájlt. További információ: [SSH használata Linuxon és Macen az Azure-on][ssh-linux].

**Virtuális gépek leállítása.** Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem.

Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha leállítja az operációs rendszer bejelentkezett keresztül, a virtuális gép le van-e állítva, de **nem** felszabadítása. lehetséges, így továbbra is fizetnie kell.

**Virtuális gépek törlése.** Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage].

Véletlen törlés megakadályozhatja, hogy egy [erőforrás zárolási] [ resource-lock] zárolását, a teljes erőforrás csoport vagy zárolási egyéni erőforrások, például egy virtuális Gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használjon [az Azure Security Center] [ security-center] való központi láthatja az Azure-erőforrások biztonsági állapotát. A Security Center a potenciális biztonsági problémákat figyeli, és biztonsági állapotát a központi telepítés átfogó képet nyújt. A Security Center / Azure-előfizetés úgy van beállítva. Biztonsági adatok gyűjtésének leírtak szerint engedélyezze a [az Azure Security Center használatába][security-center-get-started]. Adatgyűjtés engedélyezésekor a rendszer a Security Center automatikusan ellenőrzi a bármely adott előfizetésen belül létrehozott virtuális gépek.

**Javítás kezelése.** Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak. 

**Kártevőirtó.** Ha engedélyezve van, a Security Center ellenőrzi, hogy telepítve van-e a kártevőirtó szoftver. A Security Center az Azure-portálon belül a kártevőirtó szoftver telepítéséhez is használható.

**Műveletek.** Használjon [szerepköralapú hozzáférés-vezérlést (RBAC)] [ rbac] üzembe helyezett Azure-erőforrások elérésének szabályozására. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Bizonyos szerepkörök kifejezetten egy adott Azure-erőforrástípusra jellemzők. Például a virtuális gép közreműködő szerepkört is indítsa újra a virtuális gép felszabadítása, a rendszergazdai jelszó visszaállítása, hozzon létre egy új virtuális Gépet, illetve stb. Más [beépített RBAC-szerepkörök] [ rbac-roles] hasznosak lehetnek a következők ebbe az architektúrába [DevTest Labs felhasználói] [ rbac-devtest] és [ A közreműködői hálózati][rbac-network]. Egy felhasználóhoz több szerepkört is rendelhető, és létrehozhat egyéni szerepköröket a még részletesebb engedélyek érdekében.

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.   

A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás.** Ha titkosítania kell az operációs rendszert és az adatlemezeket, érdemes megfontolnia az [Azure Disk Encryption][disk-encryption] használatát. 

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez az architektúra telepítésének érhető el a [GitHub][github-folder]. Telepíti a következő:

  * Egy virtuális hálózatot egyetlen alhálózattal nevű **webes** üzemeltetni a virtuális Gépet.
  * Egy NSG két bejövő szabály az SSH és a HTTP-forgalom a virtuális géphez.
  * A legújabb verziója Ubuntu 16.04.3 virtuális gép LTS.
  * Egy minta egyéni parancsprogramok futtatására szolgáló bővítmény, amely a két adatlemezek formázza és Apache HTTP Server telepíti az Ubuntu virtuális gép.

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

1. Keresse meg a `virtual-machines\single-vm\parameters\linux` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `single-vm-v2.json` fájlt, és adjon meg egy felhasználónevet és a nyilvános SSH-kulcs az idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
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
