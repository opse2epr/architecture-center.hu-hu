---
title: "Azure virtuális gépek elosztott terhelésű futtassa a méretezhetőség és a rendelkezésre állási"
description: "Több Linux virtuális gépek futtatásához Azure a méretezhetőség és a rendelkezésre állási módját."
author: telmosampaio
ms.date: 09/07/2017
pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 0f772cdc596ea96073da3b9b2a2709ce3041889d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Virtuális gépek elosztott terhelésű a méretezhetőség és a rendelkezésre állási futtatása

A referencia-architektúrában bevált gyakorlatok futó több Linux virtuális gépeket (VM) készlete egy terheléselosztó mögött rendelkezésre állás és méretezhetőség javítása érdekében állítsa be a skála jeleníti meg. Ez az architektúra bármely állapot nélküli alkalmazások és szolgáltatások, például webkiszolgálót, nem használható, és egy építőeleme n szintű alkalmazások központi telepítéséhez. [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra

Ez az architektúra látható egy épít [a Azure Linux virtuális Gépet futtató][single vm]. A javaslatok nem ez az architektúra is vonatkozik.

Ebben az architektúrában az adott munkaterheléshez pontjain több Virtuálisgép-példányok. Egy egyetlen nyilvános IP-címet, és internetes forgalmat a virtuális gépeket, a terheléselosztó terjesztése. Ez az architektúra egyszintű alkalmazások, például az állapot nélküli webalkalmazás használható.

Az architektúra a következő részből áll:

* **Erőforráscsoport.** [*Erőforráscsoportok* ] [ resource-manager-overview] erőforrások segítségével, így élettartamát, a tulajdonos és a más feltétel alapján felügyelhetők.
* **Virtuális hálózat (VNet) és alhálózat.** Az Azure-ban minden VM egy alhálózatokra osztott virtuális hálózatban van üzembe helyezve.
* **Azure Load Balancer**. A [terheléselosztó] osztja el a Virtuálisgép-példányok bejövő Internet kéréseket. 
* **Nyilvános IP-cím**. Egy nyilvános IP-címet a terheléselosztóhoz internetes forgalom fogadására van szükség.
* **Virtuálisgép-méretezési készlet**. A [Virtuálisgép-méretezési készlet] [ vm-scaleset] üzemeltetni a munkaterhelés azonos virtuális gépek halmaza. Méretezési csoportok lehetővé teszik a méretezve, a virtuális gépek száma kívül manuálisan, vagy előre meghatározott szabályok alapján.
* **A rendelkezésre állási csoport**. A [rendelkezésre állási csoport] [ availability set] tartalmazza a virtuális gépeket, így a virtuális gépeket abban az esetben jogosult a magasabb [szolgáltatásiszint-szerződés (SLA) szolgáltatás][vm-sla]. A magasabb garantált szolgáltatási szintje alkalmazandó a rendelkezésre állási csoport tartalmaznia kell legalább két virtuális gépek. Rendelkezésre állási készletek implicit a méretezési készlet. A skála készleten kívüli virtuális gépeket hoz létre, ha meg szeretne létrehozni a rendelkezésre állási csoportot egymástól függetlenül.
* **Által kezelt lemezeken**. Azure-lemezeket kezeli a virtuális merevlemez (VHD) fájlok, a virtuális gép lemezeivel kezelése. 
* **Tárolási**. Hozzon létre egy Azure Storage-fiók a virtuális gépek diagnosztikai naplók tárolásához.

## <a name="recommendations"></a>Javaslatok

A követelmények eltérhetnek az itt leírt architektúra. Ezek a javaslatok használja kiindulópontként. 

### <a name="availability-and-scalability-recommendations"></a>Rendelkezésre állás és méretezhetőség javaslatok

A beállítás a rendelkezésre állást és méretezhetőséget, hogy használja a [virtuálisgép-méretezési csoport][vmss]. Virtuálisgép-méretezési készlet segítségével telepítheti és az azonos virtuális gépek kezelésére. Skálázási készletekben támogatási automatikus skálázás teljesítménymutatók alapján. Ha a virtuális gépeken a terhelés növekszik, további virtuális gépeket a rendszer automatikusan hozzáadja a terheléselosztót. Ha gyorsan terjessze ki a virtuális gépeket, vagy az automatikus skálázás kell módosítania, fontolja meg méretezési készlet.

Alapértelmezés szerint méretezési készlet használhatja "elhelyezésétől", ami azt jelenti, a méretezési kezdetben látja el, mint hogy kérjen további virtuális gépeket, majd törli az extra virtuális Gépekért. Ez javítja az általános sikerességi arányát, a virtuális gépek kiépítésekor. Ha nem használ [által kezelt lemezeken](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), legfeljebb 20 gépek javasoljuk a elhelyezésétől engedélyezve van, vagy legfeljebb 40 elhelyezésétől rendelkező virtuális gépek le van tiltva.

Méretezési csoportban lévő telepített virtuális gépek konfigurálása két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális Gépet, akkor kiépítése után. Ezt a módszert új Virtuálisgép-példányok hosszabb időt vehet igénybe indul el, mint egy virtuális gép nem kiterjesztésű fájl.

- Központi telepítése egy [felügyelt lemezes](/azure/storage/storage-managed-disks-overview) az egyéni lemezképet. Lehet, hogy ez a beállítás gyorsabban telepíthető. Azonban ehhez szükséges a kép naprakészen tartása.

További szempontokat lásd: [kialakítási szempontok a méretezési készlet][vmss-design].

> [!TIP]
> Minden automatikus skálázási megoldás használata esetén tesztelte éles szintű munkaterhelések jó előre.

Ha egy méretezési készlet nem használja, fontolja meg legalább használatával egy rendelkezésre állási csoportot. Hozzon létre legalább két virtuális gépek a rendelkezésre állási csoport támogatásához a [Azure virtuális gépek SLA-elérhetőséget][vm-sla]. Az Azure load balancer is szükséges, hogy elosztott terhelésű virtuális gépek ugyanabban a rendelkezésre állási csoportba tartozik.

Minden Azure-előfizetéssel rendelkezik alapértelmezett korlátokat, többek között a régiónként eltérő virtuális gépek maximális számát. A korlát növeléséhez által tervátalakítási egy támogatási kérést. További információkért lásd: [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések][subscription-limits].

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

Helyezze a virtuális gépeket ugyanazon az alhálózaton belül. Nem fedheti fel a virtuális gép közvetlenül az internethez, de ehelyett adjon egy magánhálózati IP-cím az egyes virtuális gépek. A terheléselosztó a nyilvános IP-címet használja az ügyfelek kapcsolódnak.

Ha jelentkezzen be a virtuális gépeket, a terheléselosztó mögött van szüksége, fontolja meg egy virtuális, a megerősített gazdagép/jumpbox jelentkezzen be a nyilvános IP-címmel. Majd jelentkezzen be a virtuális gépeket a jumpbox a terheléselosztó mögött. Konfigurálhatja a bejövő NAT-szabályok a terheléselosztóban ugyanerre a célra. Azonban rendelkezik egy jumpbox esetén jobb megoldás n szintű munkaterhelések, vagy több munkaterhelés üzemeltet.

### <a name="load-balancer-recommendations"></a>Load balancer javaslatok

Adja hozzá az összes virtuális gép rendelkezésre állási csoportban, a terheléselosztó a háttér-címkészlethez.

Adja meg a load balancer szabályok közvetlen hálózati forgalmat a virtuális gépekhez. Ahhoz, hogy a HTTP-forgalom, hozzon létre például egy szabályt, amely leképezhető a 80-as porton az előtér-konfigurációból a háttér-címkészlet a 80-as porton. Amikor egy ügyfél HTTP-kérelmet küld a 80-as porton, a terheléselosztó kiválasztja a háttér IP-címnek a [kivonatoló algoritmus] [ load balancer hashing] , amely tartalmazza a forrás IP-cím. Ily módon ügyfélkéréseket a virtuális gépek különböző pontjain.

Irányíthatja a forgalmat egy adott virtuális géphez, használja a NAT-szabályok. Ahhoz, hogy a virtuális gépek RDP, hozzon létre például egy külön NAT-szabály az egyes virtuális gépek. Minden egyes szabály egy eltérő portszámot kell leképezése 3389-es, az alapértelmezett port RDP. Például a "VM1", "Vm2 virtuális gépnek," 50002 port 50001 portot használja, és így tovább. Rendelje hozzá a NAT-szabályok a hálózati adaptert a virtuális gépeken.

### <a name="storage-account-recommendations"></a>Tárolási fiók javaslatok

Hozzon létre külön az Azure storage-fiókokat az egyes virtuális gépek a virtuális merevlemezek (VHD), ahhoz, hogy elérte-e a bemeneti/kimeneti műveletek száma másodpercenként elkerülése érdekében [(IOPS) vonatkozó korlátok] [ vm-disk-limits] storage-fiókok.

Azt javasoljuk, hogy használatát [által kezelt lemezeken](/azure/storage/storage-managed-disks-overview) rendelkező [prémium szintű storage][premium]. Kezelt lemezeken nincs szükség a storage-fiók. Egyszerűen adja meg, méretének és típusú lemez, és azt magas rendelkezésre állású úgy van telepítve.

Hozzon létre egy tárfiókot a diagnosztikai naplókat. Ez a tárfiók a virtuális gépek megoszthatók. Ez lehet normál lemezek nem felügyelt tárolási fiókot.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A rendelkezésre állási csoport hajt végre az alkalmazás rugalmasabb, mindkét tervezett és nem tervezett karbantartási események.

* *A tervezett karbantartások* akkor fordul elő, amikor a Microsoft frissíti az alapul szolgáló platform néha a virtuális gépek újraindítását okozza. Azure biztosítja, hogy a virtuális gépek rendelkezésre állási csoport nem mindegyike egyszerre újraindul. Legalább egy tartják mások újraindítása közben.
* *Nem tervezett karbantartás* hardverhiba esetén történik. Azure gondoskodik arról, hogy a virtuális gépek rendelkezésre állási csoportba között egynél több kiszolgálószekrény törlődnek. Ez segít a hardver meghibásodása hatásának csökkentéséhez, hálózati kimaradások, megszakítás mellett folytathatja a power és így tovább.

További információkért lásd: [virtuális gépek rendelkezésre állásának kezelése][availability set]. A következő videó is rendelkezik egy megfelelő rendelkezésre állási csoportok áttekintése: [konfigurálása a skála virtuális gépek rendelkezésre állási csoport][availability set ch9].

> [!WARNING]
> Győződjön meg arról, hogy a rendelkezésre állási csoportot, ha a virtuális gép konfigurálása. Jelenleg nincs semmilyen módon nem adja hozzá a Resource Manager virtuális gépek rendelkezésre állási készlet a virtuális gép kiépítése után.

Használ a load balancer [állapotteljesítmény] Virtuálisgép-példányok rendelkezésre állásának figyelésére. Egy mintavételt nem érhető el egy példányt a határidőn belül, ha a terheléselosztó leállítja a forgalom küldése ezt a virtuális Gépet. Azonban továbbra is a terheléselosztó mintavételi, és ha a virtuális gép ismét elérhetővé válik, a terheléselosztó folytatja-e a forgalom küldése ezt a virtuális Gépet.

Az alábbiakban néhány javaslattal terheléselosztó:

* Mintavételt tesztelheti, HTTP vagy TCP. Ha a virtuális gépek futtatása a HTTP-kiszolgáló, hozzon létre egy HTTP-vizsgálatot. Máskülönben hozzon létre egy TCP-Hálózatfigyelővel.
* HTTP-vizsgálatot adja meg a HTTP-végpont elérési útja. A mintavétel ellenőrzi, hogy az elérési út egy 200-as HTTP-válaszát. Ez lehet a legfelső szintű elérési útja ("/"), vagy egy állapotfigyelés végpontot, amely megvalósítja az egyes egyéni logika az alkalmazás állapotának ellenőrzéséhez. A végpont engedélyeznie kell a névtelen HTTP-kérelmekre.
* A mintavétel küldi a [ismert] [ health-probe-ip] 168.63.129.16 IP-címet. Győződjön meg arról, hogy a bejövő és kimenő forgalmat az IP-bármely tűzfal házirendek és a hálózati biztonsági csoport (NSG) szabályok nem tiltja le.
* Használjon [állapot-mintavételi naplók] [ health probe log] a health mintavételt állapotának megtekintéséhez. Az Azure-portál a terheléselosztók naplózásának engedélyezése. Az Azure Blob storage írja a naplókat. A naplók hány virtuális gépek megjelenítése a háttérben futó nem fordulnak elő a hálózati forgalom miatt sikertelen mintavételi válaszokat.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Több virtuális géphez fontos folyamatok automatizálása, így azok a megbízható és ismételhető. Használhat [Azure Automation] [ azure-automation] telepítési, az operációs rendszer javítását és más feladatok automatizálásához.

## <a name="security-considerations"></a>Biztonsági szempontok

Virtuális hálózatok a forgalom elkülönítési határt alkotnak, az Azure-ban. Egy virtuális hálózatot a virtuális gépek közvetlenül egy másik virtuális hálózatot a virtuális gépek nem kommunikálnak. Ugyanahhoz a virtuális gépek virtuális hálózat kommunikálhatnak, kivéve, ha hoz létre [hálózati biztonsági csoportok] [ nsg] (NSG-ket) korlátozzák a forgalmat. További információkért lásd: [Microsoft cloud services és a hálózati biztonság][network-security].

A bejövő internetes forgalmat a betöltés terheléselosztó szabályok határozzák meg, melyik forgalmat el lehet-e érni a háttérben. Azonban load balancer szabályok nem támogatják a biztonságos IP-listákat, így ha azt szeretné, hogy bizonyos nyilvános IP-címek hozzáadása a biztonságos listáját, és adja hozzá egy NSG alhálózathoz.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez

Ez az architektúra telepítésének érhető el a [GitHub][github-folder]. A méretezési, állítsa be az alábbiakban egy terheléselosztó mögött egy Vnetet, NSG és három virtuális gépeket tartalmazza:

  * Egy virtuális hálózatot egyetlen alhálózattal nevű **webes** a virtuális gépek üzemeltetéséhez használni.
  * A Virtuálisgép-méretezési készlet, amely tartalmazza a legújabb Ubuntu 16.04.3 rendszert futtató virtuális gépek LTS. Automatikus skálázás engedélyezve van.
  * Olyan terheléselosztóhoz, amely az előtérben található a Virtuálisgép-méretezési állítva.
  * Az NSG bejövő szabályok HTTP-forgalom a Virtuálisgép-méretezési való beállítása.

### <a name="prerequisites"></a>Előfeltételek

Mielőtt a saját előfizetésének telepítése a referencia-architektúrában, a következő lépésekkel kell azt.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [AzureCAT referencia architektúra] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve a számítógépre. A parancssori felület telepítéséhez kövesse az utasításokat a [Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse a [Azure építőelemeket] [ azbb] npm csomag.

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával használja az alábbi parancsok egyikét, és kövesse az utasításokat.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Az üzembe helyezéséhez azbb használatával

A minta egyetlen virtuális gép számítási feladat telepítéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `virtual-machines\multi-vm\parameters\linux` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `multi-vm-v2.json` fájlt, és adjon meg egy felhasználónév és SSH-kulcs az idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. Futtatás `azbb` a virtuális gépek telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

A minta referenciaarchitektúra telepítésével kapcsolatos további információkért látogasson el a [GitHub-tárházban][git].

<!-- Links -->
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[n-tier-linux]: ../virtual-machines-linux/n-tier.md
[n-tier-windows]: n-tier.md
[single vm]: single-vm.md
[premium]: /azure/storage/common/storage-premium-storage
[naming conventions]: /azure/guidance/guidance-naming-conventions
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[availability set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[állapotteljesítmény]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[terheléselosztó]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbook Gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[0]: ./images/multi-vm-diagram.png "Az Azure, amely magában foglalja a rendelkezésre állási készlet két virtuális gépek és a terheléselosztó virtuális Gépre kiterjedő megoldás architektúrája"
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Template-Building-Blocks-Version-2-(Linux)
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm