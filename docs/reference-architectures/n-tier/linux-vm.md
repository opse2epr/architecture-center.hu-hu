---
title: Linux virtuális gép futtatása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Ajánlott eljárások az Azure-ban futó Linux rendszerű virtuális gép.
author: telmosampaio
ms.date: 12/13/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: ec71e35bec0fa9fad604456130f8596fcf127ebb
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485655"
---
# <a name="run-a-linux-virtual-machine-on-azure"></a>Linux rendszerű virtuális gép futtatása az Azure-ban

Az Azure-beli virtuális gép (VM) szükséges, néhány további összetevők mellett a virtuális gépre, beleértve a hálózati és tárolási erőforrásokat. Ez a cikk bemutatja az ajánlott eljárások a Linux virtuális gép futtatása az Azure-ban.

![Linux rendszerű virtuális gép az Azure-ban](./images/single-vm-diagram.png)

## <a name="resource-group"></a>Erőforráscsoport

A [erőforráscsoport] [ resource-manager-overview] olyan logikai tároló, amely kapcsolódó Azure-erőforrásokat tárol. Általánosságban véve csoportba az erőforrásokat az élettartamuk alapján, és fogják kezelni őket.

A szorosan összekapcsolt, azonos életciklusú erőforrásokat helyezze egy [erőforráscsoportba][resource-manager-overview]. Az erőforráscsoportok segítségével csoportosan helyezhet üzembe és monitorozhat erőforrásokat, és a számlázási költségeket erőforráscsoportonként követheti. Erőforrások készletként, amely hasznos tesztkörnyezetek esetében is törölheti. Jelentéssel bíró erőforrásneveket adjon meg, hogy egyszerűbben megkereshesse és azonosíthassa az egyes erőforrásokat és azok szerepkörét. További információ: [Az Azure-erőforrások ajánlott elnevezési konvenciói][naming-conventions].

## <a name="virtual-machine"></a>Virtuális gép

A virtuális gépet üzembe helyezheti a közzétett rendszerképek listájáról, egy egyénileg kezelt rendszerképről, vagy egy, az Azure Blob Storage-ba feltöltött virtuálismerevlemez-fájlból (VHD).  Az Azure támogatja különböző népszerű Linux-disztribúciók, például a CentOS, a Debian, a Red Hat Enterprise, az Ubuntu és a FreeBSD futtatását. További információk: [Az Azure és a Linux][azure-linux].

Az Azure számos különböző méretét kínál a virtuális gépekhez. További információkért lásd: [az Azure-beli virtuális gépek méretei][virtual-machine-sizes]. Ha meglévő számítási feladatot helyez át az Azure-ba, kezdetnek azt a virtuálisgép-méretet válassza, amely a leginkább egyezik a helyszíni kiszolgálói méretével. Ezután mérje meg a valós számítási feladat tekintetében a Processzor, memória és a lemezek teljesítményét bemeneti/kimeneti műveletek száma másodpercenként (IOPS), és módosítsa a méretet igény szerint. 

Általánosságban elmondható válasszon egy Azure-régiót, a belső felhasználóihoz vagy ügyfeleihez legközelebb eső. Nem minden Virtuálisgép-méret érhető el minden régióban. További információ: [Szolgáltatások régiónként][services-by-region]. Az egy adott régióban elérhető virtuálisgép-méretek listájának megtekintéséhez futtassa a következő parancsot az Azure parancssori felületéről (CLI):

```azurecli
az vm list-sizes --location <location>
```

A közzétett Virtuálisgép-lemezképek kiválasztásával kapcsolatban további információt a [Linux virtuálisgép-rendszerképek megkeresésével][select-vm-image] foglalkozó részben talál.

## <a name="disks"></a>Lemezek

A legjobb adatátviteli teljesítmény érdekében javasoljuk a [Premium Storage][premium-storage] tárolást, amely SSD meghajtókon tárolja az adatokat. A költség az üzembe helyezett lemez kapacitásától függően változik. AZ IOPS és az átviteli sebesség (azaz az adatátviteli sebesség) a lemezmérettől is függ, ezért lemezek üzembe helyezésekor vegye figyelembe mindhárom tényezőt (kapacitás, IOPS és átviteli sebesség).

Azt is javasoljuk [Managed Disks][managed-disks]. A felügyelt lemezek a lemezfelügyelet egyszerűsítése a storage kezeli az Ön számára. A felügyelt lemezek nem igényelnek tárfiókot. Egyszerűen adja meg az méretű és típusú lemez és a egy magas rendelkezésre állású erőforrásként üzemel

Az operációsrendszer-lemez egy, az [Azure Storage-ban][azure-storage] tárolt VHD, ezért akkor is működik, amikor a gazdagép nem fut.  Linux virtuális gépeknél az operációsrendszer-lemez a `/dev/sda1`. Is javasolt létrehozni egy vagy több [adatlemezek][data-disk], amelyeket a perzisztens virtuális merevlemezek alkalmazásadatokhoz használt.

A létrehozott VHD-k nincsenek formázva. A lemez formázásához jelentkezzen be a virtuális gépre. A Linux felületen az adatlemezek `/dev/sdc`, `/dev/sdd` stb. formátumban jelennek meg. Az `lsblk` futtatásával listázhatja a blokkeszközöket, beleértve a lemezeket. Adatlemez használatához hozzon létre egy partíciót és egy fájlrendszert, majd csatlakoztassa a lemezt. Példa:

```bash
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

A VM egy ideiglenes lemezzel jön létre. A lemez a gazdagép egyik fizikai meghajtóján található. *Nincs* mentve az Azure Storage-ban, és előfordulhat, hogy törlődik az újraindítások és más VM-életciklusesemények során. Ez a lemez csak ideiglenes adatokat, például lapozófájlokat tárol. Linux virtuális gépeknél az ideiglenes lemez a `/dev/sdb1`, és a csatlakoztatásának helye `/mnt/resource` vagy `/mnt`.

## <a name="network"></a>Hálózat

A hálózati összetevők közé tartoznak az alábbi forrásanyagokat:

- **Virtuális hálózat**. Minden virtuális gép üzemel, amely alhálózatokra osztható virtuális hálózatban.

- **Hálózati adapter (NIC)**. A hálózati adapter teszi lehetővé a virtuális gép számára a virtuális hálózattal való kommunikációt. Ha a virtuális géphez több hálózati adapter van szüksége, vegye figyelembe, hogy a hálózati adapterek maximális száma az egyes van definiálva [Virtuálisgép-méret][vm-size-tables].

- **Nyilvános IP-cím**. Egy nyilvános IP-címre van szükség ahhoz, hogy a virtuális géppel kommunikálni lehessen – például távoli asztali kapcsolaton (RDP) keresztül. A nyilvános IP-cím lehet dinamikus vagy statikus. Alapértelmezés szerint dinamikus.

- Akkor foglaljon le [statikus IP-címet][static-ip], ha rögzített, nem módosuló IP-címre van szüksége – például ha egy „A” DNS-rekordot kell létrehoznia, vagy ha hozzá kell adnia az IP-címet a biztonságos elemek listájához.
- Létrehozhat egy teljes tartománynevet (FQDN) is az IP-címhez. Ezután a DNS-ben regisztrálhat egy, az FQDN-re mutató [CNAME rekordot][cname-record]. További információért tekintse meg a [Teljes tartománynév létrehozása az Azure Portalon][fqdn] szakaszt.

- **Hálózati biztonsági csoport (NSG)**. [Hálózati biztonsági csoportok] [ nsg] engedélyezik vagy megtagadják a hálózati forgalmat a virtuális gépek segítségével. Az NSG-k társíthatók alhálózatokhoz vagy Virtuálisgép-példányokhoz.

Minden NSG tartalmaz egy [alapértelmezett szabálykészletet][nsg-default-rules], amelyben szerepel egy minden bejövő internetes forgalmat blokkoló szabály. Az alapértelmezett szabályok nem törölhetők, azonban más szabályokkal felülírhatók. Az internetes forgalom engedélyezéséhez hozzon létre olyan szabályokat, amelyek adott portokon engedélyezik a bejövő forgalmat – például a HTTP-hez a 80-as porton.&mdash; Az SSH engedélyezéséhez adjon hozzá egy NSG-szabályt, amely engedélyezi a bejövő forgalmat a 22-es TCP-porton.

## <a name="operations"></a>Műveletek

**SSH**. Linux virtuális gép létrehozása előtt hozzon létre egy 2048 bites RSA nyilvános-titkos kulcspárt. A virtuális gép létrehozásakor használja a nyilvánoskulcs-fájlt. További információ: [SSH használata Linuxon és Macen az Azure-on][ssh-linux].

**Diagnosztikai**. Engedélyezze a megfigyelést és a diagnosztikát, beleértve az alapvető állapotmetrikákat, a diagnosztikai infrastruktúra naplófájljait és a [rendszerindítási diagnosztikát][boot-diagnostics]. A rendszerindítási diagnosztika segít diagnosztizálni a rendszerindítási hibát, ha a virtuális gép nem indítható állapotba kerül. Hozzon létre egy Azure Storage-fiókot a naplók tárolásához. Egy standard helyileg redundáns tárolási (LRS) fiók elegendő a diagnosztikai naplókhoz. További információkat [a megfigyelés és a diagnosztika engedélyezésével kapcsolatos][enable-monitoring] szakaszban találhat.

**Rendelkezésre állási**. A virtuális Gépre hatással lehet [tervezett karbantartás] [ planned-maintenance] vagy [nem tervezett üzemkimaradások][manage-vm-availability]. Annak megállapításához, hogy a virtuális gép újraindítását egy tervezett karbantartás okozta-e, használja a [virtuális gépek újraindítási naplóit][reboot-logs]. A magasabb rendelkezésre állás érdekében helyezzen üzembe több virtuális gépet egy [rendelkezésre állási csoport](/azure/virtual-machines/linux/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy). Ez a konfiguráció magasabb szintű biztosít [szolgáltatói szerződést (SLA)][vm-sla].

**Biztonsági másolatok** véletlen adatvesztés elleni védelme érdekében használja a [Azure Backup](/azure/backup/) szolgáltatást, hogy a virtuális gépek biztonsági mentése a georedundáns tárolás. Az Azure Backup alkalmazáskonzisztens biztonsági mentést biztosít.

**Virtuális gépek leállítása**. Az Azure különbséget tesz a „leállított” és a „felszabadított” állapot között. A leállított virtuális gépek után fizetni kell, a felszabadítottak után azonban nem. Az Azure Portalon a **Leállítás** gombbal szabadítható fel a virtuális gép. Ha az operációs rendszerből állítja le, amikor be van jelentkezve, azzal a virtuális gépet leállítja, de **nem** szabadítja fel, tehát továbbra is fizetnie kell a díját.

**Virtuális gépek törlése**. Ha töröl egy virtuális gépet, a VHD-k nem törlődnek. Ez azt jelenti, hogy biztonságosan törölheti a virtuális gépet anélkül, hogy adatot vesztene. A tárolásért azonban továbbra is díjat kell fizetnie. A VHD törléséhez törölje a fájlt a [Blob Storage-ból][blob-storage]. A véletlen törlés megelőzése érdekében használjon [erőforrászárat][resource-lock]. Ezzel zárolhat egy egész erőforráscsoportot, vagy egyes erőforrásokat, például egy virtuális gépet.

## <a name="security-considerations"></a>Biztonsági szempontok

Használja az [Azure Security Centert][security-center] az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez. A Security Center monitorozza a potenciális biztonsági problémákat, és átfogó képet nyújt az üzemi környezet biztonsági állapotáról. A Security Center Azure-előfizetésenként külön konfigurálandó. Biztonsági adatok gyűjtésének engedélyezése a leírtak szerint [az Azure-előfizetés a Security Center Standard verziójába történő felvételét][security-center-get-started]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

**Javítások kezelése**. Ha engedélyezve van, a Security Center ellenőrzi, hogy a biztonsági és kritikus frissítések hiányoznak-e. Használjon [Csoportszabályzat-beállításokat][group-policy] a virtuális gépen az automatikus rendszerfrissítések engedélyezéséhez.

**Kártevőirtó**. Ha engedélyezve van, a Security Center ellenőrzi, hogy van-e telepítve kártevőirtó szoftver. A Security Center segítségével telepíthet is kártevőirtó szoftvert az Azure Portalon belülről.

**Hozzáférés-vezérlés**. Használat [szerepköralapú hozzáférés-vezérlés (RBAC)] [ rbac] Azure-erőforrásokhoz való hozzáférés szabályozása. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Az Olvasó szerepkör például áttekintheti az Azure-erőforrásokat, de nem hozhatja létre, nem kezelheti és nem törölheti őket. Néhány engedélyt egy Azure-erőforrástípus jellemzőek. A Virtuális gépek közreműködője szerepkör például újraindíthat vagy felszabadíthat egy virtuális gépet, alaphelyzetbe állíthatja a rendszergazdai jelszót, létrehozhat egy új virtuális gépet, stb. Ehhez az architektúrához hasznos lehet még a [DevTest Labs-felhasználó][rbac-devtest] és a [Hálózati közreműködő][rbac-network] [beépített RBAC-szerepkör][rbac-roles].

> [!NOTE]
> Az RBAC nem korlátozza a virtuális gépre bejelentkezett felhasználó által végezhető műveleteket. Azokat az engedélyeket a vendég operációs rendszeren lévő fiók típusa határozza meg.

**Auditnaplók**. A [vizsgálati naplók][audit-logs] segítségével megtekintheti az üzembe helyezési műveleteket és más virtuálisgép-eseményeket.

**Adattitkosítás**. Használat [az Azure Disk Encryption] [ disk-encryption] , ha az operációs rendszer és az adatlemezek titkosítania kell.

## <a name="next-steps"></a>További lépések

- Egy Linux rendszerű virtuális gép, lásd: [létrehozása és kezelése a Linux rendszerű virtuális gépek az Azure CLI-vel](/azure/virtual-machines/linux/tutorial-manage-vm)
- Egy teljes körű N szintű architektúra a Linux rendszerű virtuális gépeken, lásd: [az Azure-ban az Apache Cassandra használatával N szintű Linux alkalmazás](./n-tier-cassandra.md).

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[group-policy]: /windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[managed-disks]: /azure/storage/storage-managed-disks-overview
[naming-conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
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
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
