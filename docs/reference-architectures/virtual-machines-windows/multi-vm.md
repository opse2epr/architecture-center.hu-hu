---
title: "Azure virtuális gépek elosztott terhelésű futtassa a méretezhetőség és a rendelkezésre állási"
description: "Több Windows virtuális gépek futtatásához Azure a méretezhetőség és a rendelkezésre állási módját."
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 14e7e023afd7cb7cbe0e8db8e224ba777f6fe863
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2018
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Virtuális gépek elosztott terhelésű a méretezhetőség és a rendelkezésre állási futtatása

A referencia-architektúrában több Windows virtuális gépek (VM) futtatásához bevált gyakorlatok csoportja terjedő skálán rendelkezésre állás és méretezhetőség javítása érdekében egy terheléselosztó mögött beállítása jeleníti meg. Ez az architektúra bármely állapot nélküli alkalmazások és szolgáltatások, például webkiszolgálót, nem használható és alaprendszert n szintű alkalmazások központi telepítéséhez. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Ez az architektúra épít, a [egyetlen virtuális gép referencia-architektúrában][single-vm]. Az ajánlások ebbe az architektúrába is vonatkozik.

Ebben az architektúrában az adott munkaterheléshez pontjain több Virtuálisgép-példányok. Egy egyetlen nyilvános IP-címet, és internetes forgalmat a virtuális gépeket, a terheléselosztó terjesztése. Ez az architektúra egyszintű alkalmazások, például az állapot nélküli webalkalmazás használható.

Az architektúra a következő részből áll:

* **Erőforráscsoport.** [Erőforráscsoportok] [ resource-manager-overview] erőforrások segítségével, így élettartamát, a tulajdonos vagy más feltételek felügyelhetők.
* **Virtuális hálózat (VNet) és alhálózat.** Minden Azure virtuális Gépen központilag telepítik egy Vnetet, amely több szegmentált is lehet.
* **Azure Load Balancer**. A [terheléselosztó] [ load-balancer] osztja el a Virtuálisgép-példányok bejövő Internet kéréseket. 
* **Nyilvános IP-cím**. Egy nyilvános IP-címet a terheléselosztóhoz internetes forgalom fogadására van szükség.
* **Az Azure DNS**. [Az Azure DNS] [ azure-dns] üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Virtuálisgép-méretezési készlet**. A [Virtuálisgép-méretezési készlet] [ vm-scaleset] üzemeltetni a munkaterhelés azonos virtuális gépek halmaza. Méretezési készlet lehetővé teszi a bejövő vagy kimenő tolja virtuális gépek számát manuálisan vagy automatikusan előre meghatározott szabályok alapján.
* **A rendelkezésre állási csoport**. A [rendelkezésre állási csoport] [ availability-set] tartalmazza a virtuális gépeket, így a virtuális gépeket abban az esetben jogosult a magasabb [szolgáltatásiszint-szerződés (SLA) szolgáltatás][vm-sla]. A magasabb garantált szolgáltatási szintje alkalmazandó a rendelkezésre állási csoport tartalmaznia kell legalább két virtuális gépek. Rendelkezésre állási készletek implicit a méretezési készlet. A skála készleten kívüli virtuális gépeket hoz létre, ha meg szeretne létrehozni a rendelkezésre állási csoportot egymástól függetlenül.
* **Által kezelt lemezeken**. Azure-lemezeket kezeli a virtuális merevlemez (VHD) fájlok, a virtuális gép lemezeivel kezelése. 
* **Tárolási**. Hozzon létre egy Azure Storage-fiók a virtuális gépek diagnosztikai naplók tárolásához.

## <a name="recommendations"></a>Javaslatok

A követelmények nem lehetséges, hogy teljesen igazodnak az itt leírt architektúra. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="availability-and-scalability-recommendations"></a>Rendelkezésre állás és méretezhetőség javaslatok

A beállítás a rendelkezésre állást és méretezhetőséget, hogy használja a [virtuálisgép-méretezési csoport][vmss]. Virtuálisgép-méretezési készlet segítségével telepítheti és az azonos virtuális gépek kezelésére. Skálázási készletekben támogatási automatikus skálázás teljesítménymutatók alapján. Ha a virtuális gépeken a terhelés növekszik, további virtuális gépeket a rendszer automatikusan hozzáadja a terheléselosztót. Ha gyorsan terjessze ki a virtuális gépeket, vagy az automatikus skálázás kell módosítania, fontolja meg méretezési készlet.

Alapértelmezés szerint méretezési készlet használhatja "elhelyezésétől", ami azt jelenti, a méretezési kezdetben látja el, mint hogy kérjen további virtuális gépeket, majd törli az extra virtuális Gépekért. Ez javítja az általános sikerességi arányát, a virtuális gépek kiépítésekor. Ha nem használ [által kezelt lemezeken](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), legfeljebb 20 gépek javasoljuk a elhelyezésétől engedélyezve van, és legfeljebb 40 elhelyezésétől rendelkező virtuális gépek le van tiltva.

Méretezési csoportban lévő telepített virtuális gépek konfigurálása két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális Gépet, akkor kiépítése után. Ezt a módszert új Virtuálisgép-példányok hosszabb időt vehet igénybe indul el, mint egy virtuális gép nem kiterjesztésű fájl.

- Központi telepítése egy [felügyelt lemezes](/azure/storage/storage-managed-disks-overview) az egyéni lemezképet. Lehet, hogy ez a beállítás gyorsabban telepíthető. Azonban ehhez szükséges a kép naprakészen tartása.

További szempontokat lásd: [kialakítási szempontok a méretezési készlet][vmss-design].

> [!TIP]
> Minden automatikus skálázási megoldás használata esetén tesztelte éles szintű munkaterhelések jó előre.

Ha egy méretezési készlet nem használja, fontolja meg legalább használatával egy rendelkezésre állási csoportot. Hozzon létre legalább két virtuális gépek rendelkezésre állási csoportban, támogatásához a [Azure virtuális gépek SLA-elérhetőséget][vm-sla]. Az Azure load balancer is szükséges, hogy elosztott terhelésű virtuális gépek ugyanabban a rendelkezésre állási csoportba tartozik.

Minden Azure-előfizetéssel rendelkezik alapértelmezett korlátokat, többek között a régiónként eltérő virtuális gépek maximális számát. A korlát növeléséhez által tervátalakítási egy támogatási kérést. További információkért lásd: [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések][subscription-limits].

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

Telepítse a virtuális gépeket ugyanazon az alhálózaton belül. Nem fedheti fel a virtuális gép közvetlenül az internethez, de ehelyett adjon egy magánhálózati IP-cím az egyes virtuális gépek. A terheléselosztó a nyilvános IP-címet használja az ügyfelek kapcsolódnak.

Ha jelentkezzen be a virtuális gépeket, a terheléselosztó mögött van szüksége, fontolja meg egy virtuális egy jumpbox (más néven a megerősített gazdagépen), jelentkezzen be a nyilvános IP-címmel. Majd jelentkezzen be a virtuális gépeket a jumpbox a terheléselosztó mögött. Másik lehetőségként beállíthatja a terheléselosztó bejövő hálózati cím címfordítási (NAT) szabályok. Azonban rendelkezik egy jumpbox esetén jobb megoldás n szintű munkaterhelések vagy több munkaterhelés üzemeltet.

### <a name="load-balancer-recommendations"></a>Load balancer javaslatok

Adja hozzá az összes virtuális gép rendelkezésre állási csoportban, a terheléselosztó a háttér-címkészlethez.

Adja meg a load balancer szabályok közvetlen hálózati forgalmat a virtuális gépekhez. Ahhoz, hogy a HTTP-forgalom, hozzon létre például egy szabályt, amely leképezhető a 80-as porton az előtér-konfigurációból a háttér-címkészlet a 80-as porton. Amikor egy ügyfél HTTP-kérelmet küld a 80-as porton, a terheléselosztó kiválasztja a háttér IP-címnek a [kivonatoló algoritmus] [ load-balancer-hashing] , amely tartalmazza a forrás IP-cím. Ily módon ügyfélkéréseket a virtuális gépek különböző pontjain.

Irányíthatja a forgalmat egy adott virtuális géphez, használja a NAT-szabályok. Ahhoz, hogy a virtuális gépek RDP, hozzon létre például egy külön NAT-szabály az egyes virtuális gépek. Minden egyes szabály egy eltérő portszámot kell leképezése 3389-es, az alapértelmezett port RDP. Például a "VM1", "Vm2 virtuális gépnek," 50002 port 50001 portot használja, és így tovább. Rendelje hozzá a NAT-szabályok a hálózati adaptert a virtuális gépeken.

### <a name="storage-account-recommendations"></a>Tárolási fiók javaslatok

Azt javasoljuk, hogy használatát [által kezelt lemezeken](/azure/storage/storage-managed-disks-overview) rendelkező [prémium szintű storage][premium]. Kezelt lemezeken nincs szükség a storage-fiók. Egyszerűen adja meg, méretének és típusú lemez, és azt egy magas rendelkezésre állású erőforrás van telepítve.

Ha nem felügyelt lemezt használ, hozzon létre külön az Azure storage-fiókok az egyes virtuális gépek a virtuális merevlemezek (VHD), ahhoz, hogy elérte-e a bemeneti/kimeneti műveletek száma másodpercenként elkerülése érdekében [(IOPS) vonatkozó korlátok] [ vm-disk-limits]storage-fiókok.

Hozzon létre egy tárfiókot a diagnosztikai naplókat. Ez a tárfiók a virtuális gépek megoszthatók. Ez lehet normál lemezek nem felügyelt tárolási fiókot.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A rendelkezésre állási csoport hajt végre az alkalmazás rugalmasabb, mindkét tervezett és nem tervezett karbantartási események.

* *A tervezett karbantartások* akkor fordul elő, amikor a Microsoft frissíti az alapul szolgáló platform néha a virtuális gépek újraindítását okozza. Azure biztosítja, hogy a virtuális gépek rendelkezésre állási csoport nem mindegyike egyszerre újraindul. Legalább egy tartják mások újraindítása közben.
* *Nem tervezett karbantartás* hardverhiba esetén történik. Azure gondoskodik arról, hogy a virtuális gépek rendelkezésre állási csoportba között egynél több kiszolgálószekrény törlődnek. Ez segít a hardver meghibásodása hatásának csökkentéséhez, hálózati kimaradások, megszakítás mellett folytathatja a power és így tovább.

További információkért lásd: [virtuális gépek rendelkezésre állásának kezelése][availability-set]. A következő videó is jó áttekintést nyújt a rendelkezésre állási csoportok: [konfigurálása a skála virtuális gépek rendelkezésre állási csoport][availability-set-ch9].

> [!WARNING]
> Győződjön meg arról, hogy a rendelkezésre állási csoportot, ha a virtuális gép konfigurálása. Jelenleg nincs semmilyen módon nem adja hozzá a Resource Manager virtuális gépek rendelkezésre állási készlet a virtuális gép kiépítése után.

Használ a load balancer [állapotteljesítmény] [ health-probes] Virtuálisgép-példányok rendelkezésre állásának figyelésére. Egy mintavételt nem érhető el egy példányt a határidőn belül, ha a terheléselosztó leállítja a forgalom küldése ezt a virtuális Gépet. Azonban továbbra is a terheléselosztó mintavételi, és ha a virtuális gép ismét elérhetővé válik, a terheléselosztó folytatja-e a forgalom küldése ezt a virtuális Gépet.

Az alábbiakban néhány javaslattal terheléselosztó:

* Mintavételt tesztelheti, HTTP vagy TCP. Ha a virtuális gépek futtatása a HTTP-kiszolgáló, hozzon létre egy HTTP-vizsgálatot. Máskülönben hozzon létre egy TCP-Hálózatfigyelővel.
* HTTP-vizsgálatot adja meg a HTTP-végpont elérési útja. A mintavétel ellenőrzi, hogy az elérési út egy 200-as HTTP-válaszát. Ez lehet a legfelső szintű elérési útja ("/"), vagy egy állapotfigyelés végpontot, amely megvalósítja az egyes egyéni logika az alkalmazás állapotának ellenőrzéséhez. A végpont engedélyeznie kell a névtelen HTTP-kérelmekre.
* A mintavétel küldi a [ismert IP-cím][health-probe-ip], 168.63.129.16. Győződjön meg arról, hogy a bejövő és kimenő forgalmat a IP-cím bármely tűzfal házirendek és a hálózati biztonsági csoport (NSG) szabályok nem tiltja le.
* Használjon [állapot-mintavételi naplók] [ health-probe-log] a health mintavételt állapotának megtekintéséhez. Az Azure-portál a terheléselosztók naplózásának engedélyezése. Az Azure Blob storage írja a naplókat. A naplók hány virtuális gépek megjelenítése a háttérben futó nem fordulnak elő a hálózati forgalom miatt sikertelen mintavételi válaszokat.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Több virtuális géphez fontos folyamatok automatizálása, így azok a megbízható és ismételhető. Használhat [Azure Automation] [ azure-automation] telepítési, az operációs rendszer javítását és más feladatok automatizálásához. [Azure Automation szolgáltatásbeli] [ azure-automation] az automation szolgáltatás alapul, amely nem használható a PowerShell. Példa automatizálási parancsfájlokat érhetők el a [forgatókönyvek][runbook-gallery].

## <a name="security-considerations"></a>Biztonsági szempontok

Virtuális hálózatok a forgalom elkülönítési határt alkotnak, az Azure-ban. Egy virtuális hálózatot a virtuális gépek nem tud közvetlenül kommunikálni egy másik virtuális hálózatot a virtuális gépek. Ugyanahhoz a virtuális gépek virtuális hálózat kommunikálhatnak, kivéve, ha hoz létre [hálózati biztonsági csoportok] [ nsg] (NSG-ket) korlátozzák a forgalmat. További információkért lásd: [Microsoft cloud services és a hálózati biztonság][network-security].

A bejövő internetes forgalmat a betöltés terheléselosztó szabályok határozzák meg, melyik forgalmat el lehet-e érni a háttérben. Azonban load balancer szabályok nem támogatják a biztonságos IP-listákat, így ha azt szeretné, hogy bizonyos nyilvános IP-címek hozzáadása a biztonságos listáját, és adja hozzá egy NSG alhálózathoz.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez az architektúra telepítésének érhető el a [GitHub][github-folder]. Telepíti a következő:

  * Egy virtuális hálózatot egyetlen alhálózattal nevű **webes** , amely tartalmazza a virtuális gépeket.
  * A virtuális gép méretezési csoport, amely tartalmazza a legújabb Windows Server 2016 Datacenter Edition futó virtuális gépeket. Automatikus skálázás engedélyezve van.
  * Olyan terheléselosztóhoz, amely az előtérben található a Virtuálisgép-méretezési állítva.
  * Az NSG bejövő szabályok, amelyek lehetővé teszik a HTTP-forgalom, hogy a Virtuálisgép-méretezési beállítása.

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

1. Keresse meg a `virtual-machines\multi-vm\parameters\windows` a fenti előfeltételek lépésben letöltött tárház mappát.

2. Nyissa meg a `multi-vm-v2.json` fájlt, és adjon meg egy felhasználónevet és jelszót az idézőjelek között alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Futtatás `azbb` a virtuális gépek telepítése a lent látható módon.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

A minta referenciaarchitektúra telepítésével kapcsolatos további információkért látogasson el a [GitHub-tárházban][git].

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "Az Azure, amely magában foglalja a rendelkezésre állási készlet két virtuális gépek és a terheléselosztó virtuális Gépre kiterjedő megoldás architektúrája"
