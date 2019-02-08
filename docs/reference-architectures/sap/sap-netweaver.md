---
title: SAP NetWeaver (Windows) Azure-beli virtuális gépeken AnyDB üzembe helyezése
titleSuffix: Azure Reference Architectures
description: Bevált eljárások az SAP S/4HANA környezetben futó Linux rendszerű Azure-ban magas rendelkezésre állású.
author: lbrader
ms.date: 08/03/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, SAP, Windows
ms.openlocfilehash: e866727a40551b60e74fc26878a15a5a48e69cf6
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897626"
---
# <a name="deploy-sap-netweaver-windows-for-anydb-on-azure-virtual-machines"></a>SAP NetWeaver (Windows) Azure-beli virtuális gépeken AnyDB üzembe helyezése

Ez a referenciaarchitektúra bevált eljárásokat SAP NetWeaver környezetben futó Windows Azure-ban magas rendelkezésre állású mutat be. Az adatbázis AnyDB, a bármely támogatott adatbázis-kezelő mellett az SAP HANA SAP kifejezés. Ez az architektúra, amely a szervezet igényeinek megfelelően módosíthatja adott virtuális gép (VM) méretek van telepítve.

![SAP NetWeaver (Windows) az Azure virtuális gépeken AnyDB a referencia-architektúra](./images/sap-netweaver.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

> [!NOTE]
> A referenciaarchitektúra üzembe helyezéséhez SAP-termékek és más, nem a Microsoft által gyártott termékek megfelelő licence szükséges.

## <a name="architecture"></a>Architektúra

Az architektúra a következő infrastruktúra és a kulcs szoftver összetevőkből áll.

**Virtuális hálózat**. Az Azure Virtual Network szolgáltatás Azure-erőforrások biztonságosan csatlakozik egymáshoz. Ebben az architektúrában a virtuális hálózat egy helyszíni környezetben, az agyban telepített VPN-átjárón keresztül csatlakozik egy [küllős](../hybrid-networking/hub-spoke.md). A küllő a virtuális hálózat, a SAP-alkalmazások és az Adatbázisréteg.

**Alhálózatok**. A virtuális hálózat minden szint külön alhálózatra van felosztva: alkalmazáshoz (SAP NetWeaver), adatbázis, a megosztott szolgáltatások (jumpbox) és az Active Directory.

**Virtuális gépek**. Ez az architektúra virtuális gépeket használ az alkalmazásrétegek és adatbázisszinten, az alábbiak szerint csoportosítva:

- **SAP NetWeaver**. Az alkalmazásrétegek Windows virtuális gépeket használ, és futtatja az SAP Central Services és SAP-alkalmazáskiszolgálókhoz. A virtuális gépeket, hogy futási központi szolgáltatások magas rendelkezésre állás érdekében az SIOS DataKeeper Cluster Edition által támogatott Windows Server feladatátvevő fürtként vannak konfigurálva.
- **AnyDB**. Az adatbázisszint AnyDB futtatja a forrásadatbázis, például a Microsoft SQL Server, Oracle vagy IBM DB2-höz.
- **Jumpbox**. Más néven bástyagazdagép. Ez a biztonságos virtuális gép a hálózat, amely a rendszergazdák használhatják a más virtuális gépekhez való csatlakozáshoz.
- **A Windows Server Active Directory-tartományvezérlők**. A tartományvezérlők minden olyan virtuális gépek és a tartomány felhasználói szolgálnak.

**Terheléselosztók**. [Az Azure Load Balancer](/azure/load-balancer/load-balancer-overview) példányok az alkalmazás szinten alhálózaton lévő virtuális gépek forgalom elosztására használhatók. Az adatszinten magas rendelkezésre állású elérhető beépített SAP terheléselosztók, az Azure Load Balancer vagy más mechanizmusok, attól függően, az adatbázis-kezelő használatával. További információkért lásd: [SAP NetWeaver az Azure Virtual Machines DBMS üzembe](/azure/virtual-machines/workloads/sap/dbms-guide).

**Rendelkezésre állási csoportok**. Az SAP Web Dispatcher, SAP-alkalmazáskiszolgáló és (A) SCS virtuális gépek, a szerepkörök külön vannak csoportosítva [rendelkezésre állási csoportok](/azure/virtual-machines/windows/tutorial-availability-sets), és a felhasznált szerepkörönként legalább két virtuális gépet. Ez lehetővé teszi a virtuális gépek magasabb szintű támogatásra jogosult [szolgáltatói szerződést](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA).

**Hálózati adapterek**. [Hálózati adapterek](/azure/virtual-network/virtual-network-network-interface) (NIC) a virtuális hálózaton található virtuális gépek minden kommunikáció engedélyezésére.

**Hálózati biztonsági csoportok**. Korlátozza a bejövő, kimenő, és belüli alhálózatok közötti adatforgalom a virtuális hálózatban, létrehozhat [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-k).

**Átjáró**. Az átjáró kiterjeszti a helyszíni hálózat az Azure virtual Networkhöz. [Az ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) az ajánlott Azure-szolgáltatás, amely nem a nyilvános interneten haladnak át, magánhálózati kapcsolatokat hozhat létre, de egy [Site-to-Site](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) kapcsolat is használható.

**Azure Storage**. Tartós tároláshoz a virtuális gépek virtuális merevlemez (VHD), amelyekkel [Azure Storage](/azure/storage/storage-standard-storage) megadása kötelező. Azt is használja [Felhőbeli tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) egy feladatátvételi fürt művelet végrehajtásához.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="sap-web-dispatcher-pool"></a>Az SAP Web Dispatcher-készlet

A Web Dispatcher összetevő terheléselosztóként szolgál az SAP-forgalmat az SAP-alkalmazáskiszolgálók között. Magas rendelkezésre állás a Web Dispatcher-összetevő, Azure Load Balancer segítségével a párhuzamos Web Dispatcher-beállítások megvalósításához. Web Dispatcher Ciklikus időszeleteléses konfigurációban HTTP (S)-forgalom elosztását az elérhető webes elosztás a terheléselosztók készlet használ.

Azure virtuális gépeken futó SAP NetWeaver kapcsolatos részletekért lásd: [Azure Virtual Machines tervezési és megvalósítási az SAP NetWeaver számára](/azure/virtual-machines/workloads/sap/planning-guide).

### <a name="application-servers-pool"></a>Alkalmazáskészlet-kiszolgálók

Bejelentkezési csoportokat ABAP alkalmazáskiszolgálók kezeléséhez, az SMLG tranzakció szolgál. A terheléselosztási függvényt az üzenet-kiszolgálón, a központi szolgáltatások használatával oszthatja meg a munkaterhelést SAP alkalmazáskészlet-kiszolgálók között SAPGUIs és RFC forgalmat. A magas rendelkezésre állású Central Services alkalmazás kiszolgáló kapcsolódni a fürt virtuális hálózatnevét keresztül történik.

### <a name="sap-central-services-cluster"></a>Az SAP Central Services fürt

Ez a referenciaarchitektúra a virtuális gépek központi szolgáltatást futtat az alkalmazás szinten. A központi szolgáltatások egy potenciálisan hibaérzékeny pont (SPOF), egyetlen virtuális gép telepítésekor – szokásos telepítése, ha magas rendelkezésre állás nem követelmény. Egy magas rendelkezésre állású megoldás bevezetésére, vagy egy megosztott lemezfürtök, vagy egy fájlkiszolgáló-megosztás fürt használható.

Virtuális gépek megosztott lemezfürt konfigurálásához használja [Windows Server feladatátvételi fürt](https://blogs.sap.com/2018/01/25/how-to-create-sap-resources-in-windows-failover-cluster/). [Felhőbeli tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) egy kvórum tanúsító ajánlott. A feladatátvételi fürtkörnyezetnek támogatásához [az SIOS DataKeeper Cluster Edition](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8) hajtja végre a fürt megosztott kötet függvény szerepét a fürtcsomópontok független lemezeinek replikálásával. Az Azure nem natív módon támogatja a megosztott lemezeket, és ezért az SIOS által biztosított megoldások igényel.

További információkért lásd: "3. Fontos frissítés SAP ügyfelek futó ASCS SIOS az Azure-ban a"következő [SAP alkalmazások futtatása Microsoft platformon](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/).

Kezelje a fürtszolgáltatás egy másik úgy, hogy egy fájl megosztási fürtöt a Windows Server feladatátvevő fürt megvalósítását. [SAP](https://blogs.sap.com/2018/03/19/migration-from-a-shared-disk-cluster-to-a-file-share-cluster/) nemrég módosította a szolgáltatások központi telepítési minta a /sapmnt globális könyvtárak keresztül UNC elérési út elérésére. Ez a változás [nincs szükség](https://blogs.msdn.microsoft.com/saponsqlserver/2017/08/10/high-available-ascs-for-windows-on-file-share-shared-disk-no-longer-required/) SIOS vagy más megosztott lemez megoldások központi szolgáltatások virtuális gépeken. Győződjön meg arról, hogy a /sapmnt UNC megosztást továbbra is ajánlott [magas rendelkezésre állású](https://blogs.sap.com/2017/07/21/how-to-create-a-high-available-sapmnt-share/). Ehhez a központi Services-példánytól függ használatával Windows Server feladatátvevő fürt az [kibővíthető fájlkiszolgáló](https://blogs.msdn.microsoft.com/saponsqlserver/2017/11/14/file-server-with-sofs-and-s2d-as-an-alternative-to-cluster-shared-disk-for-clustering-of-an-sap-ascs-instance-in-azure-is-generally-available/) (SOFS) és a [a közvetlen tárolóhelyek](https://blogs.sap.com/2018/03/07/your-sap-on-azure-part-5-ascs-high-availability-with-storage-spaces-direct/) (S2D) szolgáltatást a Windows Server 2016-ban.

### <a name="availability-sets"></a>Rendelkezésre állási csoportok

A rendelkezésre állási csoportok terjesztése kiszolgálók eltérő fizikai infrastruktúra és a frissítési csoportok számára szolgáltatás rendelkezésre állásának javítása. A PUT virtuális gépek ugyanazon szerepben be egy rendelkezésre állási beállítja hardvermeghibásodásokkal szemben az Azure infrastruktúra-karbantartás által okozott állásidő és kielégítése érdekében [SLA-k](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA). Két vagy több virtuális gépet egy rendelkezésre állási csoport használata javasolt.

Egy készletben lévő összes virtuális gépet ugyanarra a szerepkörre kell elvégeznie. Ne keverje a kiszolgálókat különböző szerepkörök ugyanabban a rendelkezésre állási csoportban. Például ne tegyen egy központi szolgáltatások csomópont az azonos rendelkezésre állási csoportot az a kiszolgáló.

### <a name="nics"></a>Hálózati adapterek (NIC-k)

A hagyományos helyszíni SAP-környezetekhez áthaladó üzleti forgalom a felügyeleti forgalmat leválasszanak gépenként több hálózati adapterrel (NIC) megvalósítása. Az Azure a virtuális hálózat, a szoftveresen definiált hálózati által küldött összes forgalom az azonos hálózati háló keresztül. A több hálózati adapter használatát, ezért nem szükséges. Azonban ha a szervezet igényeinek áthaladó forgalmat leválasszanak, telepítheti virtuális gépenként több hálózati adapterrel, minden egyes hálózati adapter csatlakoztatása egy másik alhálózathoz, és NSG-k segítségével különböző hozzáférés-vezérlési házirendeket kényszerítése.

### <a name="subnets-and-nsgs"></a>Alhálózat és NSG-kkel

Ez az architektúra a virtuális hálózat címtere alterületét alhálózatra. Ez a referenciaarchitektúra elsősorban az alkalmazás szint alhálózatáról összpontosít. Minden egyes alhálózathoz is társítható egy NSG-t, amely meghatározza a hozzáférési szabályzatok az alhálózat. Helyezze el alkalmazáskiszolgálók egy külön alhálózatot védelmére azokat könnyebben, mivel kezeli az alhálózat biztonsági házirendek, nem az egyes kiszolgálókon.

Amikor egy NSG-t hozzárendelik egy alhálózathoz, az alhálózaton belüli összes kiszolgálóra vonatkozik. A kiszolgálók egy alhálózat részletesen szabályozhatja az NSG-k használatával kapcsolatos további információkért lásd: [hálózati biztonsági csoportokkal a hálózati forgalom szűrése](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure).

### <a name="load-balancers"></a>Terheléselosztók

[Az SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) kezeli a egy kiszolgálókészlethez, SAP-alkalmazáskiszolgálókhoz HTTP (S)-forgalom terheléselosztásához.

Egy SAP-kiszolgálóhoz DIAG protokoll vagy függvény hívások (RFC) csatlakozó ügyfeleken az SAP grafikus felhasználói Felülettel irányuló forgalom továbbítására, a központi szolgáltatások üzenetkiszolgáló SAP-alkalmazáskiszolgáló révén a terhelés elosztja [csoportok bejelentkezési](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing), ezért nem további terhelést a terheléselosztó van szükség.

### <a name="azure-storage"></a>Azure Storage

Minden adatbázis kiszolgáló virtuális gép javasoljuk, Azure Premium Storage használatát az egyenletes olvasási/írási késés érdekében. Minden egypéldányos virtuális gép Premium Storage tárolást használ minden operációsrendszer-lemez és adatlemezek esetén lásd: [virtuális gépekre vonatkozó SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines). Ezenkívül az SAP-rendszereit, azt javasoljuk, prémium szintű [Azure Managed Disks](/azure/storage/storage-managed-disks-overview) minden esetben. Megbízhatóság, a Managed Disks kezelésére használhatók a VHD-fájlokat a lemezeket. A felügyelt lemezek győződjön meg arról, hogy a lemezek, virtuális gépek rendelkezésre állási csoporton belül elkülönített kritikus hibapontok elkerülése érdekében.

Az SAP-alkalmazáskiszolgálókhoz, beleértve a szolgáltatások központi virtuális gépeket a standard szintű Azure Storage használatával költségcsökkentés, mert az alkalmazás végrehajtása a memóriában történik, és a lemezek csak naplózás használnak. Azonban jelenleg standard szintű Storage csak minősítéssel nem felügyelt tárfiók. Mivel az alkalmazáskiszolgálókat üzemeltet adatokat, költségek minimálisra csökkentése érdekében használhatja a kisebb P4 és a P6 prémium szintű Storage-lemezeket.

Az Azure Storage is használják [Felhőbeli tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) kvórum fenntartásához egy eszközzel, az elsődleges régióban, ahol a fürtben található forrásadatok egy távoli Azure-régióban.

Az adattár biztonsági mentéséhez javasoljuk az Azure- [coolaccess szint](/azure/storage/storage-blob-storage-tiers) és [archív hozzáférési szint tárolási](/azure/storage/storage-blob-storage-tiers). Tárolási szintek költséghatékony módon a ritkán elért hosszú élettartamú adatok tárolására.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

SAP-alkalmazáskiszolgálókhoz állandó kommunikációt, az adatbázis-kiszolgálók végzi. A teljesítmény kritikus fontosságú alkalmazásokat futtató bármilyen adatbázis platformokon, beleértve az SAP HANA, érdemes lehet engedélyezni az [Írásgyorsító](/azure/virtual-machines/linux/how-to-enable-write-accelerator) napló írási késése javítása érdekében. Kiszolgálóközi kommunikáció optimalizálása érdekében használja a [gyorsított hálózati](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Vegye figyelembe, hogy a gyorsítók csak bizonyos Virtuálisgép-sorozatok érhető el.

Magas IOPS és átviteli sebessége érhető el, az általános tárolási köteten található eljárásokat [teljesítményoptimalizálás](/azure/virtual-machines/windows/premium-storage-performance) Azure tárolási elrendezést a alkalmazni. Például egy csíkozott lemez kötet létrehozása több lemez összevonása növeli a i/o-teljesítményt. A tárolóban található ritkán változó tartalmak olvasási gyorsítótárazásának engedélyezésével növelhető az adatelérés sebessége.

Az SQL, az SAP a [Top 10 kulcs szempontjai SAP-alkalmazások üzembe helyezése az Azure-ban](https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/25/top-10-key-considerations-for-deploying-sap-applications-on-azure/) blog kiváló tanácsokat az SAP számítási feladatok SQL Server az Azure storage optimalizálásához.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az SAP-alkalmazási rétegben az Azure számos különféle virtuálisgép-méretek vertikális és horizontális felskálázás kínál. A teljes listát lásd: [SAP-jegyzetnek 1928533](https://launchpad.support.sap.com/#/notes/1928533) – SAP alkalmazások az Azure-ban: Támogatott termékek és Azure Virtuálisgép-típusok. (SAP Service Marketplace-fiók szükséges az eléréshez). SAP-alkalmazáskiszolgálókhoz, és a központi szolgáltatások fürtök méretezhetők felfelé és lefelé vagy horizontálisan felskálázhat további példányok hozzáadása révén. A AnyDB adatbázis felfelé és lefelé méretezését is, de nem horizontális felskálázás. Az SAP adatbázis AnyDB tárolója nem támogatja a horizontális skálázás.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás-redundanciát a magas rendelkezésre állású infrastruktúra-megoldások az általános témát. Az olyan vállalatok, amelyek kevésbé szigorú garantált szolgáltatási szint egypéldányos Azure virtuális gépek kínálnak hasznos üzemidővel SLA-t. További információkért lásd: [Azure szolgáltatásiszint-szerződés](https://azure.microsoft.com/support/legal/sla/).

A SAP alkalmazás elosztott telepítése a rendszer replikálja az alapul szolgáló telepítés magas rendelkezésre állás. Az architektúra minden rétegében a magas rendelkezésre állás kialakítása változik.

### <a name="application-tier"></a>Alkalmazásrétegek

Magas rendelkezésre állás az SAP Web Dispatcher redundáns példányokkal érhető el. Lásd: [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) az SAP dokumentációjában.

A központi szolgáltatások magas rendelkezésre állású Windows Server feladatátvevő fürt segítségével valósul meg. Üzembe helyezett Azure-ban, a feladatátvevő fürt a fürttároló konfigurálható két módszerekkel: vagy fürtözött megosztott kötet vagy egy fájlmegosztás.

Mivel a megosztott lemezeket nem lehetséges az Azure-ban, az SIOS Datakeeper van a tartalom a fürt csomópontjaihoz csatolt független lemezek replikálásához használt, és a meghajtók absztrakt fürtként megosztott fürtkötet a kezelő számára. A megvalósítás részleteit lásd: [futó SAP ASCS fürtözése az Azure-ban](https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/).

Egy másik lehetőség az, hogy által kiszolgált fájlmegosztást a [méretezési ki Fileserver](https://blogs.msdn.microsoft.com/saponsqlserver/2017/11/14/file-server-with-sofs-and-s2d-as-an-alternative-to-cluster-shared-disk-for-clustering-of-an-sap-ascs-instance-in-azure-is-generally-available/) (SOFS). SOFS-ajánlatok a refs megosztások fürtként is használhat megosztott fürtkötet a Windows-fürt számára. Egy SOFS-fürt központi szolgáltatásokat több csomóponton között is megosztható. Jelen cikk írásakor az SOFS szolgál csak magas rendelkezésre állás kialakítása, mert az SOFS-fürthöz nem terjed ki a vész-helyreállítási támogatás biztosításához a régióban.

Magas rendelkezésre állás az SAP-alkalmazáskiszolgálók a forgalmat az alkalmazáskiszolgálók a készletben lévő terheléselosztási használatával érhető el.
Lásd: [SAP-tanúsítványok és a Microsoft Azure-on futó konfigurációk](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="database-tier"></a>Adatbázis-szint

Ez a referenciaarchitektúra azt feltételezi, hogy a forrásadatbázis on AnyDB fut – például az SQL Server, SAP ASE, IBM DB2-höz vagy az Oracle, a DBMS. Az adatbázisszint natív replikációs funkciót biztosít a replikált csomópontok közötti manuális vagy automatikus feladatátvételt.

Meghatározott adatbázis-rendszerek megvalósítási kapcsolatban lásd: [SAP NetWeaver az Azure Virtual Machines DBMS üzembe](/azure/virtual-machines/workloads/sap/dbms-guide).

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

A vészhelyreállítás (DR) meg kell tudni átadja a feladatokat egy másodlagos régióba. Minden egyes réteg más stratégiával biztosít vészhelyreállítási (DR) védelmet.

- **Kiszolgálók alkalmazásrétegek**. SAP-alkalmazáskiszolgálók nem tartalmaznak üzleti adatokat. Az Azure-ban, egy egyszerű Vészhelyreállítási stratégia lehet SAP-alkalmazáskiszolgálókhoz létrehozni a másodlagos régióba, majd leállíthatja őket. Bármilyen konfigurációmódosítás vagy kernelfrissítés az elsődleges alkalmazáskiszolgálón, hogy a módosításokat át kell másolni a virtuális gépek a másodlagos régióban. Például a kernel végrehajtható másolja a Vészhelyreállítási virtuális gépekhez. Az automatikus replikálását egy másodlagos régióba alkalmazáskiszolgálók [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) az ajánlott megoldás.

- **Központi szolgáltatások**. Az SAP alkalmazáscsoport ezen összetevője szintén nem tárol üzleti adatokat. A vészhelyreállítási régióban lévő virtuális gép futtatása a központi szolgáltatások szerepkört hozhat létre. A szinkronizálásához az elsődleges Central Services csomópont csak tartalma a /sapmnt megosztási tartalom. Is ha konfigurációmódosítás vagy kernelfrissítés történik meg a elsődleges központi szolgáltatások kiszolgálójára, azokat meg kell ismételni a közép-szolgáltatásokat futtató a vészhelyreállítási régióban lévő virtuális Gépen. A két kiszolgáló szinkronizálását, vagy az Azure Site Recovery replikálja a fürtcsomópontok, vagy egyszerűen csak egy egyszerű ütemezett másolási feladattal /sapmnt átmásolása a vészhelyreállítási régióban használhatja. További információ az egyszerű replikációs mód létrehozási, másolási és tesztelési feladatátvételi folyamatok letöltési [SAP NetWeaver: A Hyper-V és a Microsoft Azure-alapú vészhelyreállítási megoldás](https://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx), és hivatkozzon a "4.3. SAP SPOF layer (ASCS)” (SAP SPOF réteg – ASCS) fejezetet.

- **Adatbázisréteg**. DR leginkább az adatbázis a saját integrált replikációs technológiával van megvalósítva. Esetén az SQL Server például azt javasoljuk, AlwaysOn rendelkezésre állási csoport létesíteni egy távoli régióba, a replika kézi feladatátvételre tranzakció aszinkron módon replikál. Aszinkron replikáció elkerülhető az elsődleges helyen interaktív számítási feladatai teljesítményével lehet. Manuális feladatátvétel személy elemezheti a DR hatását, és döntse el, ha a DR helyről működő indokolt lehetőséget kínál.

Az Azure Site Recovery segítségével automatikusan hozhatja létre az eredeti egy teljes körűen replikált éles webhelyhez, futtatnia kell a testre szabott [üzembehelyezési szkriptek](/azure/site-recovery/site-recovery-runbook-automation). A Site Recovery először üzembe helyezi a virtuális gépek rendelkezésre állási csoportokban, majd a futtatások parancsfájlok erőforrások hozzáadásához például a terheléselosztók.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az Azure különféle funkciót nyújt [monitorozási és diagnosztikai](/azure/architecture/best-practices/monitoring) infrastruktúra átfogó. Ezenkívül fejlett monitorozását az Azure-beli virtuális gépek kezelése az Azure Operations Management Suite (OMS).

Az erőforrások és az SAP-infrastruktúra szolgáltatási teljesítményéhez SAP-alapú figyelést biztosít a [Azure SAP Enhanced Monitoring](/azure/virtual-machines/workloads/sap/deployment-guide#detailed-tasks-for-sap-software-deployment) bővítmény használatával kerül sor. Ez a bővítmény betölti az Azure monitorozási statisztikáit az SAP alkalmazásba az operációs rendszer monitorozása és a DBA Cockpit funkcióinak használata céljából.

## <a name="security-considerations"></a>Biztonsági szempontok

Az SAP egy saját felhasználófelügyeleti motorral (UME-vel) vezérli a szerepköralapú hozzáféréseket és az engedélyezést az SAP alkalmazáson belül. További információkért lásd: [biztonsági útmutató ABAP SAP NetWeaver alkalmazáskiszolgáló](https://help.sap.com/viewer/864321b9b3dd487d94c70f6a007b0397/7.4.19) és [SAP NetWeaver alkalmazás Server Java biztonsági útmutató](https://help.sap.com/doc/saphelp_snc_uiaddon_10/1.0/en-US/57/d8bfcf38f66f48b95ce1f52b3f5184/frameset.htm).

A további hálózati biztonság érdekében vegye fontolóra egy [hálózati DMZ](../dmz/secure-vnet-hybrid.md), amely egy hálózati virtuális berendezésen használatával az alhálózat elé tűzfal létrehozása az Web Dispatcher.

Az infrastruktúra biztonsága és adatokat is titkosítja az átvitt inaktív. A "Biztonsági szempontok" szakasza a [SAP NetWeaver az Azure virtuális gépeken (VM) – tervezési és megvalósítási útmutató](/azure/virtual-machines/workloads/sap/planning-guide) hálózati biztonság témájával kezdődik. Ez az útmutató adja meg azt is, hogy a tűzfalakon mely portokat kell megnyitni az alkalmazás kommunikációjának lehetővé tételéhez.

Windows virtuális gépek lemezeinek titkosításához, használhatja a [az Azure Disk Encryption](/azure/security/azure-security-disk-encryption). A Windows BitLocker funkcióját kötettitkosítást biztosít az operációs rendszer és az adatlemezeket használ. A megoldás az Azure Key Vault segítségével vezérelheti és felügyelheti a lemeztitkosítási kulcsokat és titkos kulcsokat a key vault-előfizetés is használható. A virtuális gépek lemezein található, inaktív adatok az Azure Storage-ban titkosítva vannak.

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

- [A Microsoft Platform Blog futó SAP-alkalmazások](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Azure közösségi támogatás](https://azure.microsoft.com/support/community/)
- [Az SAP közösségi](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Tekintse át az alábbiakat érdemes [Azure példaforgatókönyvek](/azure/architecture/example-scenario) , amelyek bemutatják, hogy egyes technológiákat használó adott megoldások:

- [Az SAP számítási feladatok futtatása Azure-beli Oracle-adatbázis használata](/azure/architecture/example-scenario/apps/sap-production)
- [Az SAP-feladatokat az Azure-ban fejlesztési és tesztelési környezetek](/azure/architecture/example-scenario/apps/sap-dev-test)

<!-- links -->

[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx
