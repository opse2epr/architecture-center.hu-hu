---
title: SAP S/4HANA, a Linux rendszerű virtuális gépek az Azure-ban
description: Bevált eljárások az SAP S/4HANA környezetben futó Linux rendszerű Azure-ban magas rendelkezésre állású.
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: 9635de73ec431e0ac678e4008e0c4835796d47ad
ms.sourcegitcommit: 86d86d71e392550fd65c4f76320d7ecf0b72e1f6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/06/2018
ms.locfileid: "37864504"
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>SAP S/4HANA, a Linux rendszerű virtuális gépek az Azure-ban

Ez a referenciaarchitektúra bevált eljárásokat S/4hana-t, amely támogatja a vészhelyreállítást az Azure-ban magas rendelkezésre állású környezetben futó mutat be. Ez az architektúra, amely a szervezet igényeinek megfelelően módosíthatja adott virtuális gép (VM) méretek van telepítve. 

![](./images/sap-s4hana.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

> [!NOTE] 
> A referenciaarchitektúra üzembe helyezéséhez SAP-termékek és más, nem a Microsoft által gyártott termékek megfelelő licence szükséges.

## <a name="architecture"></a>Architektúra
 
Ez a referenciaarchitektúra egy nagyvállalati szintű, termelési szintű rendszer ismerteti. Az üzleti igényeinek megfelelően kell, ez a konfiguráció egyetlen virtuális gép lehet szűkíteni. Azonban a következő összetevők szükségesek:

**Virtuális hálózat**. A [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) szolgáltatás Azure-erőforrások biztonságosan csatlakozik egymáshoz. Ebben az architektúrában a virtuális hálózat egy helyszíni környezetben telepített a hubon, az átjárón keresztül csatlakozik egy [küllős topológia](../hybrid-networking/hub-spoke.md). A küllő a virtuális hálózat, az SAP-alkalmazásokhoz használható.

**Alhálózatok**. A virtuális hálózaton van felosztva, külön [alhálózatok](/azure/virtual-network/virtual-network-manage-subnet) az egyes szintek: átjáró, az alkalmazás, az adatbázis és a megosztott szolgáltatások. 

**Virtuális gépek**. Ez az architektúra az alkalmazásrétegek és adatbázisszinten, az alábbiak szerint csoportosítva Linux rendszerű virtuális gépek használja:

- **Alkalmazásrétegek**. A Fiori előtérbeli kiszolgálókészlet, SAP Web Dispatcher-készlet, alkalmazáskészletet kiszolgáló és az SAP Central Services fürt tartalmaz. Magas rendelkezésre álláshoz központi szolgáltatások az Azure-beli Linuxos virtuális gépek magas rendelkezésre állású hálózati fájlrendszer (NFS) szolgáltatás kötelező megadni.
- **NFS-fürt**. Ez az architektúra használ egy [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) futtató Linux-fürt, SAP-rendszerek között megosztott adatok tárolására. Erre a központi fürtre több SAP-rendszeren keresztül is megoszthatók. Az NFS-szolgáltatás magas rendelkezésre állás érdekében a megfelelő magas rendelkezésre állás kiterjesztése a kijelölt Linux-disztribúció szolgál.
- **SAP HANA**. Az adatbázisszint egy fürt két vagy több Linux rendszerű virtuális gépek magas rendelkezésre állás használja. HANA rendszer replikációs (HSR) tartalma replikálni az elsődleges és másodlagos HANA rendszerek között történik. Linux-fürtszolgáltatás rendszerhibák észlelését, és automatikus feladatátvételi elősegítésére szolgál. A storage-alapú vagy felhőalapú szintaxiskiemeléshez mechanizmus a sikertelen rendszer elkülönített, vagy állítsa le a fürt maszkolt feltétel elkerülése érdekében győződjön meg arról, hogy használható.
- **Jumpbox**. Más néven bástyagazdagép. Ez a biztonságos virtuális gép a hálózat, amely a rendszergazdák használhatják a más virtuális gépekhez való csatlakozáshoz. Windows vagy Linux rendszeren futtatható. Használjon egy Windows jumpbox böngészési kényelmi célokat szolgál, HANA vezérlőpultot vagy a HANA Studio felügyeleti eszközök használata esetén.

**Terheléselosztók**. Mindkét SAP beépített terheléselosztók és [Azure Load Balancer](/azure/load-balancer/load-balancer-overview) segítségével magas rendelkezésre ÁLLÁS elérése érdekében. Az Azure Load Balancer-példányok segítségével osztja el a forgalmat az alkalmazás szinten alhálózatot a virtuális gépek.

**Rendelkezésre állási csoportok**. Virtuális gépek összes készletek és-fürtök (Web Dispatcher, az SAP alkalmazáskiszolgálók, központi szolgáltatások, az NFS és HANA) külön vannak csoportosítva [rendelkezésre állási csoportok](/azure/virtual-machines/windows/tutorial-availability-sets), és a felhasznált szerepkörönként legalább két virtuális gépet. Ez lehetővé teszi a virtuális gépek magasabb szintű támogatásra jogosult [szolgáltatói szerződést](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA). 

**Hálózati adapterek**. [Hálózati adapterek](/azure/virtual-network/virtual-network-network-interface) (NIC) a virtuális hálózaton található virtuális gépek minden kommunikáció engedélyezésére.

**Hálózati biztonsági csoportok**. Korlátozza a bejövő, kimenő, és belüli alhálózatok közötti adatforgalom a virtuális hálózatban [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-k) használhatók.

**Átjáró**. Az átjáró kiterjeszti a helyszíni hálózat az Azure virtual Networkhöz. [Az ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) az ajánlott Azure-szolgáltatás, amely nem a nyilvános interneten haladnak át, magánhálózati kapcsolatokat hozhat létre, de egy [Site-to-Site](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) kapcsolat is használható. 

**Azure Storage**. Tartós tároláshoz a virtuális gépek virtuális merevlemez (VHD), amelyekkel [Azure Storage](/azure/storage/) megadása kötelező. 

## <a name="recommendations"></a>Javaslatok

Ez az architektúra egy kisméretű éles környezetre vállalati telepítését ismerteti. A központi telepítés üzleti igényei alapján eltérnek. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="virtual-machines"></a>Virtual machines (Virtuális gépek)

A kiszolgáló alkalmazáskészletekhez és -fürtök állítsa be a virtuális gépek igényei alapján számát. A [tervezési és megvalósítási útmutató Azure virtuális gépek](/azure/virtual-machines/workloads/sap/planning-guide) kitérnek hamarosan SAP NetWeaver futtatása virtuális gépeken, de az adatokat az SAP S/4hana-t is vonatkozik.

SAP részleteit támogatja az Azure-beli virtuálisgép-típusok és átviteli sebességre vonatkozó mérőszámok (SAP), lásd: [SAP Megjegyzés 1928533](https://launchpad.support.sap.com/#/notes/1928533). 

### <a name="sap-web-dispatcher-pool"></a>Az SAP Web Dispatcher-készlet

A Web Dispatcher összetevő terheléselosztóként szolgál az SAP-forgalmat az SAP-alkalmazáskiszolgálók között. Magas rendelkezésre állás a Web Dispatcher-összetevő, a párhuzamos Web Dispatcher-telepítő megvalósítása HTTP (S)-forgalom elosztását az elérhető webes elosztás a engedélyezve ciklikus multiplexelési konfigurációjának az Azure Load Balancer szolgál háttér-címkészlet. 

### <a name="fiori-front-end-server"></a>Fiori előtér-kiszolgáló

A Fiori előtér-kiszolgáló használ egy [NetWeaver átjáró](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true). A kisméretű környezetek betölthetők a Fiori-kiszolgálón. Nagyméretű környezetekben NetWeaver átjáró egy különálló kiszolgálón is üzembe helyezhetők a Fiori előtérbeli kiszolgálókészlet elé.

### <a name="application-servers-pool"></a>Alkalmazáskészlet-kiszolgálók

Bejelentkezési csoportokat ABAP alkalmazáskiszolgálók kezeléséhez, az SMLG tranzakció szolgál. A terheléselosztási függvényt az üzenet-kiszolgálón, a központi szolgáltatások használatával oszthatja meg a munkaterhelést SAP alkalmazáskészlet-kiszolgálók között SAPGUIs és RFC forgalmat. A magas rendelkezésre állású Central Services alkalmazás kiszolgáló kapcsolódni a fürt virtuális hálózatnevét keresztül történik. Ezzel elkerülhető a kell módosítani az alkalmazásprofil server központi Services-kapcsolat a helyi feladatátvétel után. 

### <a name="sap-central-services-cluster"></a>Az SAP Central Services fürt

Központi szolgáltatások egyetlen virtuális gép is telepíthető, ha magas rendelkezésre állás nem követelmény. Azonban az egyetlen virtuális gép lesz egy potenciálisan hibaérzékeny pont (SPOF) az SAP-környezet számára. A magas rendelkezésre állású szolgáltatások központi telepítések NFS magas rendelkezésre állású fürt és a egy magas rendelkezésre állású Central Services fürt szolgálnak.

### <a name="nfs-cluster"></a>NFS-fürt

Az NFS-fürt a csomópont közötti replikálás DRBD (elosztott replikált blokk eszköz) használható.

### <a name="availability-sets"></a>Rendelkezésre állási csoportok

A rendelkezésre állási csoportok terjesztése kiszolgálók eltérő fizikai infrastruktúra és a frissítési csoportok számára szolgáltatás rendelkezésre állásának javítása. A PUT virtuális gépek ugyanazon szerepben be egy rendelkezésre állási beállítja hardvermeghibásodásokkal szemben az Azure infrastruktúra-karbantartás által okozott állásidő és kielégítése érdekében [SLA-k](https://azure.microsoft.com/support/legal/sla/virtual-machines). Két vagy több virtuális gépet egy rendelkezésre állási csoport használata javasolt.

Egy készletben lévő összes virtuális gépet ugyanarra a szerepkörre kell elvégeznie. Ne keverje a kiszolgálókat különböző szerepkörök ugyanabban a rendelkezésre állási csoportban. Például ne tegyen ASCS csomópont az azonos rendelkezésre állási csoportot az a kiszolgáló.

### <a name="nics"></a>Hálózati adapterek (NIC-k)

A hagyományos helyszíni SAP-környezetünk áthaladó üzleti forgalom a felügyeleti forgalmat leválasszanak gépenként több hálózati adapterrel (NIC) megvalósítása. Az Azure a virtuális hálózat, a szoftveresen definiált hálózati által küldött összes forgalom az azonos hálózati háló keresztül. A több hálózati adapter használatát, ezért nem szükséges. Azonban ha a szervezet igényeinek áthaladó forgalmat leválasszanak, telepítheti virtuális gépenként több hálózati adapterrel, minden egyes hálózati adapter csatlakoztatása egy másik alhálózathoz, és NSG-k segítségével különböző hozzáférés-vezérlési házirendeket kényszerítése.

### <a name="subnets-and-nsgs"></a>Alhálózat és NSG-kkel

Ez az architektúra a virtuális hálózat címtere alterületét alhálózatra. Minden egyes alhálózathoz is társítható egy NSG-t, amely meghatározza a hozzáférési szabályzatok az alhálózat. Helyezze el alkalmazáskiszolgálók egy külön alhálózatot védelmére azokat könnyebben, mivel kezeli az alhálózat biztonsági házirendek, nem az egyes kiszolgálókon.

Amikor egy NSG-t hozzárendelik egy alhálózathoz, majd érvényes az alhálózaton belüli összes kiszolgálón. A kiszolgálók egy alhálózat részletesen szabályozhatja az NSG-k használatával kapcsolatos további információkért lásd: [hálózati biztonsági csoportokkal a hálózati forgalom szűrése](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/).

Lásd még: [tervezéssel és kialakítással VPN-átjáró](/azure/vpn-gateway/vpn-gateway-plan-design).

### <a name="load-balancers"></a>Terheléselosztók

[Az SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) kezeli a például egy kiszolgálókészlethez, SAP-alkalmazáskiszolgálókhoz Fiori-stílus alkalmazások HTTP (S)-forgalom terheléselosztásához. 

Egy SAP-kiszolgálóhoz DIAG vagy függvény hívások (RFC) csatlakozó ügyfeleken az SAP grafikus felhasználói Felülettel irányuló forgalom továbbítására, a központi szolgáltatás üzenetkiszolgáló SAP-alkalmazáskiszolgáló révén a terhelés elosztja [csoportok bejelentkezési](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing), ezért semmilyen további terheléselosztó nem szükséges. 

### <a name="azure-storage"></a>Azure Storage

Az adatbázis-kiszolgáló virtuális gépek az Azure Premium Storage használatát javasoljuk. A Premium storage az egyenletes olvasási/írási késést biztosít. Az operációsrendszer-lemezek és az adatlemezek egypéldányos virtuális gépek Premium Storage használatával kapcsolatos részletekért lásd: [virtuális gépekre vonatkozó SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

Az összes éles SAP-rendszereit, javasoljuk a prémium szintű [Azure Managed Disks](/azure/storage/storage-managed-disks-overview). A felügyelt lemezek segítségével kezelheti a VHD-fájlok, lemezek, megbízhatóság hozzáadása. Akkor is győződjön meg arról, hogy a rendelkezésre állási csoporton belül a virtuális gépek lemezei elkülönített kritikus hibapontok elkerülése érdekében.

Az SAP-alkalmazáskiszolgálókhoz, beleértve a szolgáltatások központi virtuális gépeket a standard szintű Azure Storage használatával költségcsökkentés, mert az alkalmazás végrehajtása a memóriában történik, és pedig csak naplózásra használja a lemezek. Azonban jelenleg standard szintű Storage csak minősítéssel nem felügyelt tárfiók. Mivel az alkalmazáskiszolgálókat üzemeltet adatokat, költségek minimálisra csökkentése érdekében használhatja a kisebb P4 és a P6 prémium szintű Storage-lemezeket.

Az adattár biztonsági mentéséhez javasoljuk az Azure- [ritka elérésű hozzáférési szint tárolási és/vagy archív hozzáférési szint tárolási](/azure/storage/storage-blob-storage-tiers). Tárolási szintek költséghatékony módon, hogy a kevésbé gyakran használt, hosszú élettartamú adatok tárolására.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

SAP-alkalmazáskiszolgálókhoz állandó kommunikációt, az adatbázis-kiszolgálók végzi. A HANA database virtuális gépek, érdemes lehet engedélyezni az [Írásgyorsító](/azure/virtual-machines/linux/how-to-enable-write-accelerator) napló írási késése javítása érdekében. Kiszolgálóközi kommunikáció optimalizálása érdekében használja a [gyorsított hálózati](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Vegye figyelembe, hogy a gyorsítók csak bizonyos Virtuálisgép-sorozatok érhető el.

Magas IOPS és átviteli sebessége érhető el, az általános tárolási köteten található eljárásokat [teljesítményoptimalizálás](/azure/virtual-machines/linux/premium-storage-performance) Azure tárolási elrendezést a alkalmazni. Például egy csíkozott lemez kötet létrehozása több lemez összevonása növeli a i/o-teljesítményt. A tárolóban található ritkán változó tartalmak olvasási gyorsítótárazásának engedélyezésével növelhető az adatelérés sebessége. Teljesítmény-követelményeivel kapcsolatos részletekért lásd: [SAP-jegyzetnek 1943937 - hardverkonfiguráció-ellenőrzési eszközről](https://launchpad.support.sap.com/#/notes/1943937) (SAP Service Marketplace-en a hozzáféréshez szükséges fiók).

## <a name="scalability-considerations"></a>Méretezési szempontok

Az SAP-alkalmazási rétegben az Azure számos különféle virtuálisgép-méretek vertikális és horizontális felskálázás kínál. A teljes listát lásd: [SAP Megjegyzés 1928533](https://launchpad.support.sap.com/#/notes/1928533) – SAP alkalmazások az Azure-on: támogatott termékek és Azure-beli Virtuálisgép-típusok (SAP Service Marketplace-en a hozzáféréshez szükséges fiók). Folyamatos tanúsítása több virtuálisgép-típust, méretezhetők felfelé és lefelé az azonos felhőbeli üzemelő példány. 

Az adatbázisrétegben Ez az architektúra futtatja a HANA-beli virtuális gépeken. Ha a számítási feladatok meghaladja a maximális Virtuálisgép-méretet, Microsoft által [nagyméretű Azure-példányokon](/azure/virtual-machines/workloads/sap/hana-overview-architecture) az SAP Hana-hoz. Ezek a fizikai kiszolgálók elhelyezve egy Microsoft Azure certified adatközpont és a jelen cikk írásakor, adja meg a memória-kapacitás egyetlen példányra, akár 20 TB. Többcsomópontos konfigurációval a teljes memóriakapacitás akár 60 TB.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás-redundanciát a magas rendelkezésre állású infrastruktúra-megoldások az általános témát. Az olyan vállalatok, amelyek kevésbé szigorú garantált szolgáltatási szint egypéldányos Azure virtuális gépek kínálnak hasznos üzemidővel SLA-t. További információkért lásd: [Azure szolgáltatásiszint-szerződés](https://azure.microsoft.com/support/legal/sla/).

A SAP alkalmazás elosztott telepítése a rendszer replikálja az alapul szolgáló telepítés magas rendelkezésre állás. Az architektúra minden rétegében a magas rendelkezésre állás kialakítása változik. 

### <a name="application-tier"></a>Alkalmazásrétegek

- Web Dispatcher. Magas rendelkezésre állású redundáns Web Dispatcher-példányokkal érhető el. Lásd: [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) az SAP dokumentációjában.
- Fiori-kiszolgálók. Magas rendelkezésre állású van terheléselosztásával belüli kiszolgálók készletét.
- Központi szolgáltatások. Magas rendelkezésre állása érdekében központi szolgáltatások Azure-beli Linuxos virtuális gépeken a megfelelő magas rendelkezésre állás-bővítmény linuxhoz a kiválasztott terjesztési szolgál, és a magas rendelkezésre állású NFS gazdagépek DRBD tárolási fürt.
- Alkalmazáskiszolgálók. A magas rendelkezésre állás az adatforgalom terheléselosztásával érhető el alkalmazáskiszolgálók egy csoportjában.

### <a name="database-tier"></a>Adatbázis-szint

Ez a referenciaarchitektúra álló két Azure-beli virtuális gépek magas rendelkezésre állású SAP HANA database rendszer mutatja be. Az adatbázisszint natív rendszer replikációs funkciót biztosít a replikált csomópontok közötti manuális vagy automatikus feladatátvételt:

- Kézi feladatátvételre egynél több HANA-példány üzembe helyezése és használata a HANA System replikációs (HSR).
- Az automatikus feladatátvételhez használja a HSR- és Linux magas rendelkezésre állási bővítmény (HAE) is a Linux-disztribúció. Linux HAE a HANA-erőforrásokat, a fürt szolgáltatást biztosítja az események észlelése és replikálásával segít a vállalatnak a kifogástalan állapotú csomópontra errant szolgáltatások a feladatátvételt. 

Lásd: [SAP-tanúsítványok és a Microsoft Azure-on futó konfigurációk](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok
Minden egyes réteg más stratégiával biztosít vészhelyreállítási (DR) védelmet.

- **Kiszolgálók alkalmazásrétegek**. SAP-alkalmazáskiszolgálók nem tartalmaznak üzleti adatokat. Az Azure-ban, egy egyszerű Vészhelyreállítási stratégia lehet SAP-alkalmazáskiszolgálókhoz létrehozni a másodlagos régióba, majd leállíthatja őket. Bármilyen konfigurációmódosítás vagy kernelfrissítés az elsődleges alkalmazáskiszolgálón esetén a másodlagos régióban lévő virtuális gépek a módosításokat kell alkalmazni. Például másolja át az SAP-kernel végrehajtható a Vészhelyreállítási virtuális gépeket. Az automatikus replikálását egy másodlagos régióba alkalmazáskiszolgálók [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) az ajánlott megoldás. Ez a cikk írásakor, az ASR nem még támogatja a gyorsított hálózati konfigurációs beállítás, a replikáció az Azure-beli virtuális gépeken.

- **Központi szolgáltatások**. Az SAP alkalmazáscsoport ezen összetevője szintén nem tárol üzleti adatokat. A központi szolgáltatások szerepkör futtatásához a másodlagos régió virtuális Gépet hozhat létre. A szinkronizálásához az elsődleges Central Services csomópont csak tartalma a /sapmnt megosztási tartalom. Is ha konfigurációmódosítás vagy kernelfrissítés történik meg a elsődleges központi szolgáltatások kiszolgálójára, akkor meg kell ismételni a virtuális gép központi szolgáltatásokat futtató másodlagos régióban. A két kiszolgáló szinkronizálását, vagy az Azure Site Recovery segítségével a fürtcsomópontok replikálni, vagy egyszerűen csak egy egyszerű ütemezett másolási feladattal segítségével /sapmnt másolja a DR oldalra. További részleteket tartalmaz a build másolási és tesztelési feladatátvételi folyamatok letöltése [SAP NetWeaver: Hyper-V és a Microsoft Azure-alapú vészhelyreállítási megoldás létrehozása](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx), és szakaszban 4.3-as, "SAP SPOF layer (ASCS)." Ez a tanulmány a Windows rendszerű NetWeaver vonatkozik, de egyenértékű konfigurációnak felel meg a Linux rendszerre is létrehozhat. Központi szolgáltatások használata [Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview) replikálni a fürtcsomópontok és a tárolás. Linux esetén hozzon létre egy három csomópontos geo-fürt egy magas rendelkezésre állási bővítmény használatával. 

- **Az SAP adatbázis-szintű**. HANA által támogatott replikációs HSR használja. Egy helyi, kétcsomópontos magas rendelkezésre állású telepítés mellett HSR replikációs többrétegű, ahol egy külön Azure-régióban egy harmadik csomóponton nem része a fürt külső jogi személyként funkcionál, és regisztrálja a másodlagos replikának a fürtözött HSR-pár, támogatja a replikációs cél. A replikációs lánckapcsolt ez alkotnak. A feladatátvételt, hogy a DR-csomópont manuális folyamat során a rendszer.

Az Azure Site Recovery segítségével automatikusan hozhat létre az eredeti egy teljes körűen replikált éles webhelyhez, futtatnia kell a testre szabott [üzembehelyezési szkriptek](/azure/site-recovery/site-recovery-runbook-automation). A Site Recovery először üzembe helyezi a virtuális gépek rendelkezésre állási csoportokban, majd a futtatások parancsfájlok erőforrások hozzáadásához például a terheléselosztók. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az SAP HANA rendelkezik egy biztonsági mentési funkció teszi lehetővé, használhatja az alapul szolgáló Azure-infrastruktúra. Biztonsági mentése az Azure-beli virtuális gépeken futó SAP HANA-adatbázis, a SAP HANA pillanatképe és az Azure-tároló pillanatképe is használhatók a biztonsági mentési fájlok konzisztenciájának biztosítása. További információkért lásd: [biztonsági mentési útmutató az SAP Hana az Azure Virtual Machinesben](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide) és a [Azure Backup szolgáltatás – gyakori kérdések](/azure/backup/backup-azure-backup-faq). Csak a HANA egyetlen tároló üzemelő példányai támogatják az Azure storage-pillanatkép.

### <a name="identity-management"></a>Identitáskezelés

Minden szinten egy központosított identitáskezelési rendszerének az erőforrásokhoz való hozzáférés vezérlése:

- Azure-erőforrások keresztül hozzáférést biztosítanak [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-control-what-is) (RBAC). 
- Hozzáférés engedélyezése az Azure-beli virtuális gépek LDAP, Azure Active Directory, a Kerberos vagy egy másik rendszeren keresztül. 
- Támogatási hozzáférését magukat az alkalmazásokban az SAP biztosít, vagy használjon szolgáltatásokon keresztül [OAuth 2.0 és az Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code). 

### <a name="monitoring"></a>Figyelés

Az Azure különféle funkciót nyújt [monitorozási és diagnosztikai](/azure/architecture/best-practices/monitoring) infrastruktúra átfogó. Emellett az Azure-beli virtuális gépek (Linux vagy Windows) fejlett monitorozását az Azure Operations Management Suite (OMS) végzi. 

Az erőforrások és az SAP-infrastruktúra szolgáltatási teljesítményéhez SAP-alapú figyelést biztosít a [Azure SAP Enhanced Monitoring](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca) bővítmény használatával kerül sor. Ez a bővítmény betölti az Azure monitorozási statisztikáit az SAP alkalmazásba az operációs rendszer monitorozása és a DBA Cockpit funkcióinak használata céljából. Kötelező futtatásának előfeltétele, hogy az SAP az Azure SAP enhanced monitoring. További információkért lásd: [SAP Megjegyzés 2191498](https://launchpad.support.sap.com/#/notes/2191498) – "SAP használata Linux az Azure-ral: figyelés fokozott."

## <a name="security-considerations"></a>Biztonsági szempontok

Az SAP egy saját felhasználófelügyeleti motorral (UME-vel) vezérli a szerepköralapú hozzáféréseket és az engedélyezést az SAP alkalmazáson belül. További információkért lásd: [SAP HANA biztonsági – áttekintés](https://archive.sap.com/documents/docs/DOC-62943) (SAP Service Marketplace-fiók szükséges a hozzáféréshez.)

A további hálózati biztonság érdekében vegye fontolóra egy [hálózati DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid), melyik egy hálózati virtuális készüléket használ egy tűzfal elé Web Dispatcher és Fiori előtér-kiszolgáló készletek az alhálózat létrehozásához.

Az infrastruktúra biztonsága és adatokat is titkosítja az átvitt inaktív. A "Biztonsági szempontok" szakasza a [SAP NetWeaver az Azure Virtual Machines – tervezési és megvalósítási útmutató](/azure/virtual-machines/workloads/sap/planning-guide) hálózati biztonság témájával kezdődik, és alkalmazza az S/4hana-t. Ez az útmutató adja meg azt is, hogy a tűzfalakon mely portokat kell megnyitni az alkalmazás kommunikációjának lehetővé tételéhez. 

Linux rendszerű IaaS virtuális gépek lemezeinek titkosításához, használhatja a [az Azure Disk Encryption](/azure/security/azure-security-disk-encryption). A Linux DM-Crypt funkcióját kötettitkosítást biztosít az operációs rendszer és az adatlemezeket használ. A megoldás az Azure Key Vault segítségével vezérelheti és felügyelheti a lemeztitkosítási kulcsokat és titkos kulcsokat a key vault-előfizetés is használható. A virtuális gépek lemezein található, inaktív adatok az Azure Storage-ban titkosítva vannak.

Az SAP HANA inaktív adatainak titkosításához javasoljuk az SAP HANA natív titkosítási technológiájának használatát. 

> [!NOTE]
> Ugyanazon a kiszolgálón ne használjon a HANA inaktív adatok titkosítása az Azure Disk Encryption. Hana csak a HANA adattitkosítást használnia.

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

- [A Microsoft Platform Blog futó SAP-alkalmazások](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Azure közösségi támogatás](https://azure.microsoft.com/support/community/)
- [Az SAP közösségi](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)

[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx
