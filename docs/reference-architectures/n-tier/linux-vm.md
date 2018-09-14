---
title: Linux virtuális gép futtatása az Azure-ban
description: A Linux virtuális gépek futtatása az Azure-ban a skálázhatóságot, a rugalmasságot, a kezelhetőséget és a biztonságot is figyelembe véve.
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: b53db016a594bace880aaac4e16f0586fe3057b1
ms.sourcegitcommit: 25bf02e89ab4609ae1b2eb4867767678a9480402
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/14/2018
ms.locfileid: "45584731"
---
# <a name="run-a-linux-vm-on-azure"></a>Linux virtuális gép futtatása az Azure-ban

Ez a cikk bemutatja a bevált eljárásokat futtatásához egy Linux rendszerű virtuális gép (VM) az Azure-ban. Javaslatokat tartalmaz a VM, valamint a hálózatkezelési és tárolási összetevők üzembe helyezéséhez. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![[0]][0]

## <a name="components"></a>Összetevők

Az Azure virtuális Gépekhez szükséges, néhány további összetevők mellett a virtuális gépre, beleértve a hálózati és tárolási erőforrásokat.

* **Erőforráscsoport.** A [erőforráscsoport] [ resource-manager-overview] olyan logikai tároló, amely kapcsolódó Azure-erőforrásokat tárol. Általánosságban véve csoportba az erőforrásokat az élettartamuk alapján, és fogják kezelni őket. 

* **VM**. A virtuális gépet üzembe helyezheti a közzétett rendszerképek listájáról, egy egyénileg kezelt rendszerképről, vagy egy, az Azure Blob Storage-ba feltöltött virtuálismerevlemez-fájlból (VHD). Az Azure támogatja különböző népszerű Linux-disztribúciók, például a CentOS, a Debian, a Red Hat Enterprise, az Ubuntu és a FreeBSD futtatását. További információk: [Az Azure és a Linux][azure-linux].

* **A Managed Disks**. [Az Azure Managed Disks] [ managed-disks] lemezfelügyelet egyszerűsítése a storage kezeli az Ön számára. Az operációsrendszer-lemez egy, az [Azure Storage-ban][azure-storage] tárolt VHD, ezért akkor is működik, amikor a gazdagép nem fut. Linux virtuális gépeknél az operációsrendszer-lemez a `/dev/sda1`. Is javasolt létrehozni egy vagy több [adatlemezek][data-disk], amelyeket a perzisztens virtuális merevlemezek alkalmazásadatokhoz használt. 

* **Ideiglenes lemez.** A VM egy ideiglenes lemezzel jön létre. A lemez a gazdagép egyik fizikai meghajtóján található. *Nincs* mentve az Azure Storage-ban, és előfordulhat, hogy törlődik az újraindítások és más VM-életciklusesemények során. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol. Linux virtuális gépeknél az ideiglenes lemez a `/dev/sdb1`, és a csatlakoztatásának helye `/mnt/resource` vagy `/mnt`.

* **Virtuális hálózat (VNet).** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve.

* **Hálózati adapter (NIC)**. A hálózati adapter teszi lehetővé a virtuális gép számára a virtuális hálózattal való kommunikációt.

* **Nyilvános IP-cím.** Egy nyilvános IP-címre van szükség ahhoz, hogy a virtuális géppel kommunikálni lehessen – például SSH-n keresztül.

* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

* **Hálózati biztonsági csoport (NSG)**. [Hálózati biztonsági csoportok] [ nsg] engedélyezik vagy megtagadják a hálózati forgalmat a virtuális gépek segítségével. Az NSG-k társíthatók alhálózatokhoz vagy Virtuálisgép-példányokhoz.

* **Diagnosztika.** A diagnosztikai naplózás létfontosságú a virtuális gép kezeléséhez és hibaelhárításához.

## <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Az Azure számos különböző méretét kínál a virtuális gépekhez. További információkért lásd: [az Azure-beli virtuális gépek méretei][virtual-machine-sizes]. Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Ezután mérje meg a valós számítási feladat teljesítményét a CPU, a memória és a lemez másodpercenkénti bemeneti/kimeneti műveletei (IOPS) figyelembe vételével, és módosítsa a méretet, ha szükséges. Ha több hálózati adapterre van szükség a virtuális géphez, vegye figyelembe, hogy a hálózati adapterek maximális száma minden [virtuálisgép-mérethez][vm-size-tables] külön van meghatározva.

Általánosságban elmondható válasszon egy Azure-régiót, a belső felhasználóihoz vagy ügyfeleihez legközelebb eső. Azonban nem minden virtuálisgép-méret érhető el minden régióban. További információ: [Szolgáltatások régiónként][services-by-region]. Az egy adott régióban elérhető virtuálisgép-méretek listájának megtekintéséhez futtassa a következő parancsot az Azure parancssori felületéről (CLI):

```
az vm list-sizes --location <location>
```

A közzétett Virtuálisgép-lemezképek kiválasztásával kapcsolatban további információt a [Linux virtuálisgép-rendszerképek megkeresésével][select-vm-image] foglalkozó részben talál.

Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. A rendszerindítási diagnosztika segít diagnosztizálni a rendszerindítási hibát, ha a virtuális gép nem indítható állapotba kerül. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.  

## <a name="disk-and-storage-recommendations"></a>A lemezre és a tárolásra vonatkozó javaslatok

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. A költség az üzembe helyezett lemez kapacitásától függően változik. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség). 

Azt is javasoljuk [Managed Disks][managed-disks]. A felügyelt lemezek nem igényelnek tárfiókot. Egyszerűen adja meg a méretet és a lemez típusát, és az magas rendelkezésre állású erőforrásként lesz üzembe helyezve.

Adjon hozzá egy vagy több adatlemezt. A létrehozott VHD-k nincsenek formázva. A lemez formázásához jelentkezzen be a virtuális gépre. A Linux felületen az adatlemezek `/dev/sdc`, `/dev/sdd` stb. formátumban jelennek meg. Az `lsblk` futtatásával listázhatja a blokkeszközöket, beleértve a lemezeket. Adatlemez használatához hozzon létre egy partíciót és egy fájlrendszert, majd csatlakoztassa a lemezt. Példa:

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

Hozzon létre egy tárfiókot a diagnosztikai naplók tárolására. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz.

> [!NOTE]
> Ha nem használ felügyelt lemezeket, hozzon létre külön Azure storage-fiókot minden virtuális Géphez annak érdekében, hogy elkerülje a virtuális merevlemezek (VHD-k) tárolására a [IOPS-korlátok] [ vm-disk-limits] tárfiókok esetében. Vegye figyelembe a tárfiók teljes i/o-korlátját. További információkért lásd a [virtuálisgép-lemez korlátaival kapcsolatos][vm-disk-limits] szakaszt.

## <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

* Akkor foglaljon le [statikus IP-címet][static-ip], ha rögzített, nem módosuló IP-címre van szüksége – például ha egy A rekordot kell létrehoznia a DNS-ben, vagy ha hozzá kell adnia az IP-címet a biztonságos elemek listájához.&mdash;
* Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információért tekintse meg a [Teljes tartománynév létrehozása az Azure Portalon][fqdn] szakaszt. Használhatja az [Azure DNS-t][azure-dns] vagy más DNS-szolgáltatást.

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash;

Az SSH engedélyezéséhez adjon hozzá egy NSG-szabályt, amely engedélyezi a bejövő forgalmat a 22-es TCP-porton.

## <a name="scalability-considerations"></a>Méretezési szempontok

A virtuális gépeket a [virtuális gép méretének módosításával][vm-resize] skálázhatja. A horizontális felskálázáshoz helyezzen két vagy több virtuális gépet egy terheléselosztó mögé. További információkért lásd: a [referencia-architektúra N szintű](./n-tier-cassandra.md).

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy rendelkezésre állási csoportban. Ez magasabb szintű [szolgáltatói szerződést (SLA-t)][vm-sla] biztosít.

A virtuális gépre hatással lehet egy [tervezett karbantartás][planned-maintenance] vagy egy [nem tervezett karbantartás][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs].

A normál műveletek során történő véletlen (például hiba miatti) adatvesztés elleni védelem érdekében érdemes időponthoz kötött biztonsági mentéseket is megvalósítani [blob-pillanatképekkel][blob-snapshot] vagy más eszközzel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

**Erőforráscsoportok.** A szorosan összekapcsolt, azonos életciklusú erőforrásokat helyezze egy [erőforráscsoportba][resource-manager-overview]. Az erőforráscsoportok segítségével csoportosan helyezhet üzembe és monitorozhat erőforrásokat, és a számlázási költségeket erőforráscsoportonként követheti. Az erőforrásokat törölheti is készletenként. Ez nagyon hasznos tesztkörnyezetek esetében. Jelentéssel bíró erőforrásneveket adjon meg, hogy egyszerűbben megkereshesse és azonosíthassa az egyes erőforrásokat és azok szerepkörét. További információ: [Az Azure-erőforrások ajánlott elnevezési konvenciói][naming-conventions].

**SSH**. Linux virtuális gép létrehozása előtt hozzon létre egy 2048 bites RSA nyilvános-titkos kulcspárt. A virtuális gép létrehozásakor használja a nyilvánoskulcs-fájlt. További információ: [SSH használata Linuxon és Macen az Azure-on][ssh-linux].

**Virtuális gépek leállítása.** Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem. Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha az operációs rendszerből állítja le, amikor be van jelentkezve, azzal a virtuális gépet leállítja, de **nem** szabadítja fel, tehát továbbra is fizetnie kell a díját.

**Virtuális gépek törlése.** Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage]. A véletlen törlés megelőzése érdekében használjon [erőforrászárat][resource-lock]. Ezzel zárolhat egy egész erőforráscsoportot, vagy egyes erőforrásokat, például egy virtuális gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használja az [Azure Security Centert][security-center] az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez. A Security Center monitorozza a potenciális biztonsági problémákat, és átfogó képet nyújt az üzemi környezet biztonsági állapotáról. A Security Center Azure-előfizetésenként külön konfigurálandó. Engedélyezze a biztonsági adatok gyűjtését [az Azure Security Center rövid útmutatójában leírtak szerint][security-center-get-started]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

**Javítások kezelése.** Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak-e. 

**Kártevőirtó.** Ha engedélyezve van, a Security Center ellenőrzi, hogy van-e telepítve kártevőirtó szoftver. A Security Center segítségével telepíthet is kártevőirtó szoftvert az Azure Portalon belülről.

**Műveletek.** A [szerepköralapú hozzáférés-vezérléssel (RBAC)][rbac] szabályozható az üzembe helyezett Azure-erőforrásokhoz való hozzáférés. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Bizonyos szerepkörök kifejezetten egy adott Azure-erőforrástípusra jellemzők. A Virtuális gépek közreműködője szerepkör például újraindíthat vagy felszabadíthat egy virtuális gépet, alaphelyzetbe állíthatja a rendszergazdai jelszót, létrehozhat egy új virtuális gépet, stb. Ehhez az architektúrához hasznos lehet még a [DevTest Labs-felhasználó][rbac-devtest] és a [Hálózati közreműködő][rbac-network] [beépített RBAC-szerepkör][rbac-roles]. Egy felhasználóhoz több szerepkört is rendelhető, és létrehozhat egyéni szerepköröket a még részletesebb engedélyek érdekében.

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.   

A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás.** Ha titkosítania kell az operációs rendszert és az adatlemezeket, érdemes megfontolnia az [Azure Disk Encryption][disk-encryption] használatát. 

**A DDoS protection**. Ajánlott engedélyezni az [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), amely biztosítja, hogy egy virtuális hálózatban található erőforrások további DDoS-kockázatcsökkentést. Alapszintű DDoS elleni védelem részeként az Azure platform automatikusan engedélyezve van, noha a DDoS Protection Standard kockázatcsökkentési képességeket biztosít, amelyek kifejezetten az Azure virtuális hálózati erőforrások, amelyek ideálisak.  

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Központi telepítés érhető el az [GitHub][github-folder]. A következőket helyezi üzembe:

  * Egy virtuális hálózat egyetlen alhálózattal (neve **web**), amely a virtuális gépet üzemelteti.
  * Egy két bejövő szabállyal rendelkező NSG, amely engedélyezi az SSH- és a HTTP-forgalmat a virtuális gép felé.
  * Egy virtuális gép, amely az Ubuntu 16.04.3 LTS legújabb verzióját futtatja.
  * Egy egyéni minta szkriptbővítmény, amely formázza a két adatlemezt, és üzembe helyezi az Apache HTTP Servert az Ubuntu virtuális gépen.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

5. Hozzon létre ssh-kulcs. További információkért lásd: [létrehozása és a nyilvános és titkos ssh-kulcs használata Linux rendszerű virtuális gépekhez az Azure-ban](/azure/virtual-machines/linux/mac-create-ssh-keys).

### <a name="deploy-the-solution-using-azbb"></a>A megoldás üzembe helyezése az azbb használatával

1. Navigáljon az előző lépésekben letöltött adattár `virtual-machines/single-vm/parameters/linux` mappájához.

2. Nyissa meg a `single-vm-v2.json` fájlt, és adjon meg egy felhasználónevet és a nyilvános SSH-kulcsot az idézőjelek között, majd mentse a fájlt.

  ```bash
  "adminUsername": "<your username>",
  "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
  ```

3. Futtassa az `azbb` parancsot az alábbi módon a minta virtuális gép telepítéséhez.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

Az üzembe helyezés ellenőrzéséhez futtassa a következő Azure CLI-parancsot a virtuális gép nyilvános IP-cím található:

```bash
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

Ha ezt a címet egy böngészőben keresse meg az alapértelmezett Apache2 kezdőlapjának kell megjelennie.

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
[managed-disks]: /azure/storage/storage-managed-disks-overview
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Egyetlen Linux rendszerű virtuális gép az Azure-ban"
