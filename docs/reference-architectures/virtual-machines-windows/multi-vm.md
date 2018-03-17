---
title: "Elosztott terhelésű virtuális gépek üzemeltetése a méretezhetőség és a rendelkezésre állás érdekében"
description: "Több Windows rendszerű virtuális gép futtatása az Azure-ban a jobb méretezhetőség és a rendelkezésre állás érdekében."
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: 5bbaed67d5eaa8a195b35d4f79a33122bbd77a23
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/17/2018
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Elosztott terhelésű virtuális gépek üzemeltetése a méretezhetőség és a rendelkezésre állás érdekében

Ez a referenciaarchitektúra bevált eljárásokat mutat be, amelyekkel a méretezhetőség és a rendelkezésre állás javítása érdekében több Windows rendszerű virtuális gép (VM) futtatható egy terheléselosztó mögötti méretezési csoportban. Ez az architektúra bármilyen állapot nélküli számítási feladathoz használható (például webkiszolgálókhoz), és alapul szolgál az N szintű alkalmazások üzembe helyezéséhez. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Ez az architektúra az [egyetlen virtuális géppel rendelkező referenciaarchitektúrára][single-vm] épít. Az arra vonatkozó javaslatok erre az architektúrára is érvényesek.

Ebben az architektúrában a számítási feladatok több virtuálisgép-példányon oszlanak el. Az architektúra egyetlen nyilvános IP-címet használ, és az internetes forgalom terheléselosztón keresztül jut el a virtuális gépekhez. Ez az architektúra egyrétegű alkalmazásokhoz, például állapot nélküli webalkalmazásokhoz használható.

Az architektúra a következő összetevőkből áll:

* **Erőforráscsoport.** Az [erőforráscsoportok][resource-manager-overview] az erőforrások csoportosítására használhatók, így élettartamuk, tulajdonosuk vagy egyéb jellemzőik alapján kezelhetők.
* **Virtuális hálózat (VNet) és alhálózat.** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve.
* **Azure Load Balancer**. A [terheléselosztó][load-balancer] elosztja a beérkező internetes kérelmeket a virtuálisgép-példányok között. 
* **Nyilvános IP-cím**. Ahhoz, hogy a terheléselosztó fogadni tudja az internetes forgalmat, egy nyilvános IP-címre van szükség.
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Virtuálisgép-méretezési csoport**. A [Virtuálisgép-méretezési csoport] [ vm-scaleset] egymással megegyező virtuális gépek csoportja, amely egy adott számítási feladatot üzemeltet. A méretezési csoportok lehetővé teszik, hogy a virtuális gépek száma manuálisan, vagy előre meghatározott szabályok szerint automatikusan fel- vagy leskálázható legyen.
* **Rendelkezésre állási csoport**. A [rendelkezésre állási csoport][availability-set] magában foglalja a virtuális gépeket, így azok magasabb szintű [szolgáltatói szerződésre (SLA-ra)][vm-sla] jogosultak. A magasabb szintű szolgáltatói szerződés érvénybe lépéséhez a rendelkezésre állási csoportnak legalább két virtuális gépet kell tartalmaznia. A rendelkezésre állási csoportok a méretezési csoportokkal járnak. Ha méretezési csoporton kívül hoz létre virtuális gépeket, külön létre kell hoznia a rendelkezésre állási csoportot.
* **Felügyelt lemezek**. Az Azure Managed Disks szolgáltatás kezeli a virtuális gépek lemezeihez tartozó virtuális merevlemezek (VHD) fájljait. 
* **Tárhely**. Hozzon létre egy Azure Storage-fiókot a virtuális gépek diagnosztikai naplóinak tárolására.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="availability-and-scalability-recommendations"></a>Rendelkezésre állási és méretezhetőségi javaslatok

A rendelkezésre állás és a méretezhetőség érdekében használható egyik lehetőség a [virtuálisgép-méretezési csoport][vmss]. A virtuálisgép-méretezési csoportok segítségével azonos virtuális gépek csoportját hozhatja létre és kezelheti. A méretezési csoportok támogatják a teljesítménymetrikák alapján történő automatikus skálázást. Ahogy a terhelés növekszik a virtuális gépeken, a rendszer további virtuális gépeket ad a terheléselosztóhoz. Fontolja meg a méretezési csoportok használatát, ha virtuális gépek gyors horizontális felskálázására vagy automatikus méretezésre van szüksége.

Alapértelmezés szerint a méretezési csoportok „túlzott mértékű kiépítést” használnak, ami azt jelenti, hogy a méretezési csoport kezdetben az Ön által igényeltnél több virtuális gépet oszt ki, majd törli a nem szükséges példányokat. Ez javítja az általános sikeresség arányát a virtuális gépek kiépítésekor. Ha nem használ [felügyelt lemezeket](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), túlzott mértékű kiépítéssel beállított tárfiókonként 20-nál több virtuális gép használata nem ajánlott, a túlzott mértékű kiépítés letiltása mellett pedig legfeljebb 40 virtuális gép ajánlott.

A méretezési csoportokban üzembe helyezett virtuális gépek konfigurálásának két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális gépet annak kiépítése után. Ezzel a módszerrel az új virtuálisgép-példányok indítása több időt vehet igénybe, mint a bővítmények nélküli virtuális gépeké.

- Üzembe helyezhet egy [felügyelt lemezt](/azure/storage/storage-managed-disks-overview) egy egyéni rendszerképpel. Ez a módszer gyorsabban kivitelezhető. Azonban ebben az esetben naprakészen kel tartania a rendszerképet.

További szempontokért lásd: [Kialakítási szempontok a méretezési csoportokhoz][vmss-design].

> [!TIP]
> Bármilyen automatikus méretezési megoldás használata esetén jóval előre tesztelje azt az éles környezetre jellemző mennyiségű számítási feladattal.

Ha nem használ méretezési csoportot, fontolja meg legalább egy rendelkezésre állási csoport használatát. Hozzon létre legalább két virtuális gépet a rendelkezésre állási csoportban az [Azure-beli virtuális gépekre vonatkozó rendelkezésre állási SLA][vm-sla] támogatásához. Az Azure-terheléselosztó további követelménye, hogy az elosztott terhelésű virtuális gépek ugyanabba a rendelkezésre állási csoportba tartozzanak.

Minden Azure-előfizetésre alapértelmezett korlátozások vonatkoznak. Ilyen például a régiónként alkalmazható virtuális gépek maximális száma. Támogatási kérelem kitöltésével megnövelheti ezt a korlátot. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][subscription-limits].

### <a name="network-recommendations"></a>Hálózatokra vonatkozó javaslatok

A virtuális gépeket ugyanabban az alhálózatban helyezze üzembe. Ne engedélyezze a virtuális gépek elérését közvetlenül az internetről – ehelyett adjon mindegyiknek privát IP-címet. Az ügyfelek a terheléselosztó nyilvános IP-címével kapcsolódnak.

Ha a terheléselosztó mögötti virtuális gépekre kell bejelentkeznie, javasolt egy virtuális gép jumpboxként (más néven bástyagazdagépként) való hozzáadása egy olyan nyilvános IP-címmel, amelyre be tud jelentkezni. Ezután jelentkezzen be a terheléselosztó mögötti virtuális gépekbe a jumpboxról. Alternatív lehetőségként konfigurálhatja a terheléselosztó bejövő hálózati címfordítási (NAT-) szabályait. Azonban a jumpbox jobb megoldás, ha N szintű számítási feladatokat vagy több számítási feladatot kíván futtatni.

### <a name="load-balancer-recommendations"></a>Terheléselosztóra vonatkozó javaslatok

Adja hozzá a rendelkezésre állási csoportban található összes virtuális gépet a terheléselosztó háttércímkészletéhez.

Adja meg a terheléselosztó a virtuális gépek felé irányuló közvetlen hálózati forgalomra vonatkozó szabályait. Például a HTTP-forgalom engedélyezéséhez hozzon létre egy szabályt, amely hozzárendeli a 80-as portot az előtérbeli konfigurációból a háttércímkészletben található 80-as porthoz. Amikor egy ügyfél HTTP-kérelmet küld a 80-as port felé, a terheléselosztó kiválaszt egy háttérbeli IP-címet egy [kivonatoló algoritmus][load-balancer-hashing] használatával, amely tartalmazza a forrás IP-címét. Így a kérelmek megoszlanak az összes virtuális gép között.

Ahhoz, hogy az adatforgalmat egy meghatározott virtuális gépre irányítsa, használjon NAT-szabályokat. Ha például szeretné engedélyezni az RDP-vel való kapcsolódást a virtuális gépekhez, hozzon létre egy külön NAT-szabályt minden egyes virtuális gép számára. Minden egyes szabálynak hozzá kell rendelnie egy meghatározott portszámot a 3389-es porthoz, azaz az RDP-hez használt alapértelmezett porthoz. Például használja az 50001-es portot a „VM1” virtuális géphez, az 50002-est a „VM2”-höz, és így tovább. A NAT-szabályokat a virtuális gépek hálózati adaptereihez rendelje hozzá.

### <a name="storage-account-recommendations"></a>Tárfiókokra vonatkozó javaslatok

Javasoljuk a [felügyelt lemezek](/azure/storage/storage-managed-disks-overview) és a [prémium szintű tároló][premium] használatát. A felügyelt lemezek nem igényelnek tárfiókot. Egyszerűen adja meg a méretet és a lemez típusát, és az magas rendelkezésre állású erőforrásként lesz üzembe helyezve.

Ha nem felügyelt lemezeket használ, hozzon létre külön Azure Storage-fiókot minden virtuális géphez a virtuális merevlemezek (VHD-k) tárolására, hogy elkerülje az [IOPS-korlátok][vm-disk-limits] (másodpercenkénti bemeneti/kimeneti műveletek) elérését a tárfiókban.

Hozzon létre egy tárfiókot a diagnosztikai naplók számára. Ezt a tárfiókot az összes virtuális gép megoszthatja. A fiók lehet általános lemezeket használó nem felügyelt tárfiók.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

A rendelkezésre állási csoport rugalmasabbá teszi alkalmazását mind a tervezett, mind a nem tervezett karbantartási eseményekkel szemben.

* *Tervezett karbantartás* akkor fordul elő, amikor a Microsoft frissíti az alapul szolgáló platformot. Ez olykor a virtuális gépek újraindításával jár. Az Azure gondoskodik arról, hogy egy rendelkezésre állási csoporton belüli virtuális gépek ne egy időben induljanak újra. Legalább egy mindig fut, amíg a többi újraindítása folyamatban van.
* *Nem tervezett karbantartás* hardverhiba esetén történik. Az Azure gondoskodik arról, hogy egy rendelkezésre állási csoporton belüli virtuális gépek több kiszolgálószekrényben helyezkedjenek el. Ez csökkenti az esetleges hardverhibák, hálózati kimaradások, áramkimaradások és hasonlók hatását.

További információk: [Virtuális gépek rendelkezésre állásának kezelése][availability-set]. A következő videó áttekintést nyújt a rendelkezésre állási csoportokról: [Hogyan konfigurálhatók rendelkezésre állási csoportok a virtuális gépek méretezéséhez?][availability-set-ch9].

> [!WARNING]
> Mindenképp konfigurálja a rendelkezésre állási csoportot a virtuális gép kiépítésekor. Jelenleg nem lehet Resource Manager-alapú virtuális gépet hozzáadni egy rendelkezésre állási csoporthoz a virtuális gép kiépítése után.

A terheléselosztó [állapotmintákat][health-probes] használ a virtuálisgép-példányok rendelkezésre állásának monitorozásához. Ha a mintavétel nem ér el egy példányt egy bizonyos időkorláton belül, a terheléselosztó nem irányít több forgalmat az adott virtuális gép felé. A terheléselosztó ezután is folytatja a mintavételezést, és amint a virtuális gép újra elérhetővé válik, a terheléselosztó ismét elkezd forgalmat irányítani felé.

Az alábbiakban néhány javaslat olvasható a terheléselosztó állapot-mintavételére vonatkozóan:

* A mintavételek a HTTP-t vagy a TCP-t tesztelhetik. Ha a virtuális gépek HTTP-kiszolgálót futtatnak, HTTP-mintavételt hozzon létre. Egyéb esetben TCP-mintavételt hozzon létre.
* A HTTP-mintavétel esetében adjon meg elérési utat egy HTTP-végponthoz. A mintavétel 200-as HTTP-választ vár erről a megadott elérési útról. Ez lehet a gyökérútvonal („/”) vagy egy állapotmonitorozó végpont, amely valamilyen egyéni logikát alkalmaz az alkalmazás állapotának ellenőrzésére. A végpontnak engedélyeznie kell a névtelen HTTP-kérelmeket.
* A mintavétel [ismert IP-címről érkezik:][health-probe-ip] 168.63.129.16. Győződjön meg arról, hogy nem tiltja a bejövő és kimenő forgalmat erről az IP-címről valamelyik tűzfalszabályzatban vagy NSG-szabályban.
* Az állapotminták állapotának megtekintéséhez használjon [állapotminta-naplókat][health-probe-log]. Engedélyezze a naplózást az Azure Portalon minden egyes terheléselosztóra vonatkozóan. A naplókat a rendszer az Azure Blob Storage-ba írja. A naplók megmutatják, hány háttérbeli virtuális gép nem fogad hálózati forgalmat a meghiúsult mintavételi válaszok miatt.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Több virtuális gép használata esetén fontos a folyamatok automatizálása azok megbízhatósága és megismételhetősége érdekében. Az [Azure Automation][azure-automation] használatával automatizálhatja az üzembe helyezést, az operációs rendszer javításait és az egyéb feladatokat. Az [Azure Automation][azure-automation] egy ehhez használható PowerShell-alapú automatizálási szolgáltatás. Az automatizálási példaszkriptek a [Runbookkatalógusban][runbook-gallery] érhetők el.

## <a name="security-considerations"></a>Biztonsági szempontok

A virtuális hálózatok forgalomelkülönítési határok az Azure-ban. Egy adott virtuális hálózatban található virtuális gépek nem képesek közvetlenül kommunikálni egy másik virtuális hálózat gépeivel. Az azonos virtuális hálózatban elhelyezkedő virtuális gépek képesek a kommunikációra, hacsak [hálózati biztonsági csoportokat] [ nsg] (NSG-ket) nem hoz létre a forgalom korlátozására. További információ: [A Microsoft felhőszolgáltatásai és hálózati biztonság][network-security].

A bejövő internetes forgalom esetében a terheléselosztó szabályai határozzák meg, milyen forgalom érheti el a háttérrendszert. A terheléselosztó szabályai azonban nem támogatják a biztonságos IP-címek listázását, ezért ha fel kíván venni bizonyos nyilvános IP-címeket a biztonságos címek listájára, adjon hálózati biztonsági csoportot az alhálózathoz.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. A következőket helyezi üzembe:

  * Egy virtuális hálózatot egyetlen, **web** nevű alhálózattal, amely tartalmazza a virtuális gépeket.
  * Olyan virtuálisgép-méretezési csoport, amely a Windows Server 2016 Datacenter Edition legfrissebb verzióját futtató virtuális gépeket tartalmaz. Az automatikus méretezés engedélyezve van.
  * Egy terheléselosztót, amely a virtuálisgép-méretezési csoport előtt helyezkedik el.
  * Egy hálózati biztonsági csoportot bejövő forgalomra vonatkozó szabályokkal, amelyek engedélyezik a HTTP-forgalmat a virtuálisgép-méretezési csoport felé.

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

1. Navigáljon az előfeltételekre vonatkozó előző lépésekben letöltött adattár `virtual-machines\multi-vm\parameters\windows` mappájához.

2. Nyissa meg a `multi-vm-v2.json` fájlt, és adjon meg egy felhasználónevet és egy jelszót az idézőjelek között az alább látható módon, majd mentse a fájlt.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Az `azbb` futtatásával helyezze üzembe a virtuális gépet az alább látható módon.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

A mintául szolgáló referenciaarchitektúra üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.

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
[0]: ./images/multi-vm-diagram.png "Több Azure-beli virtuális gépet tartalmazó megoldás architektúrája, amely egy két virtuális gépet tartalmazó rendelkezésre állási csoportból és egy terheléselosztóból áll"
