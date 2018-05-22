---
title: Az Azure virtuális gépeken AnyDB telepítése SAP NetWeaver (Windows)
description: Eljárások az SAP S/4HANA környezetben futó Linux Azure magas rendelkezésre állású bizonyítása.
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: 0efe3e78d9e1809fdab52044b75e432742786b79
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/21/2018
---
# <a name="deploy-sap-netweaver-windows-for-anydb-on-azure-virtual-machines"></a>Az Azure virtuális gépeken AnyDB telepítése SAP NetWeaver (Windows)

A referencia-architektúrában az SAP NetWeaver környezetben futó Windows Azure magas rendelkezésre állású bevált gyakorlatok csoportja jeleníti meg. Az adatbázis AnyDB, bármely támogatott adatbázis-kezelő SAP HANA mellett az SAP kifejezés. Ez az architektúra van szükség az adott virtuális gép (VM), amelyek módosíthatók a szervezet igényeinek megfelelően van telepítve.

![](./images/sap-s4hana.png)
 
> [!NOTE] 
> A referencia-architektúrában megfelelően SAP termékek alkalmazástelepítéshez szükséges megfelelő licencelési SAP-termékek és más nem Microsoft-technológiák.

## <a name="architecture"></a>Architektúra
Az architektúra a következő infrastruktúra és a kulcs szoftver összetevőkből áll.

**Virtuális hálózat**. Az Azure Virtual Network szolgáltatás Azure-erőforrások biztonságosan csatlakozik egymáshoz. Ebben a rendszerben a virtuális hálózat csatlakozik egy helyszíni környezetben keresztül VPN-átjáró telepítése a központban egy [hub-küllős](../hybrid-networking/hub-spoke.md). A küllős a virtuális hálózat az SAP-alkalmazások és adatbázis-rétegből használt.

**Alhálózatok**. A virtuális hálózat különálló alhálózatokat az egyes rétegekhez oszlik: alkalmazás (SAP NetWeaver), a adatbázis, a megosztott szolgáltatások (a jumpbox) és az Active Directory.
    
**Virtuális gépek**. Ez az architektúra virtuális gépek használ az alkalmazás réteg és adatbázis-rétegből, az alábbiak szerint csoportosítva:

- **SAP NetWeaver**. Az alkalmazás használja a Windows virtuális gépek és központi SAP-szolgáltatások és az SAP alkalmazáskiszolgálók futtat. A virtuális gépeket, hogy futási központi szolgáltatás magas rendelkezésre állás érdekében SIOS DataKeeper fürt kiadása támogatja a Windows Server feladatátvevő fürtként van-e konfigurálva.
- **AnyDB**. Az adatbázis-rétegből AnyDB a forrásadatbázis, például a Microsoft SQL Server, Oracle vagy IBM DB2 futtatja.
- **Jumpbox**. Más néven bástyagazdagép. Ez a biztonságos virtuális gép, amely a rendszergazdák használhatják a virtuális számítógépekhez kapcsolódni a hálózaton.
- **Windows Server Active Directory-tartományvezérlők**. A tartományvezérlők használ az összes virtuális gépek és a tartomány felhasználóinak.

**Terheléselosztók**. Mindkét SAP beépített terheléselosztók és [Azure terheléselosztó](/azure/load-balancer/load-balancer-overview) magas rendelkezésre ÁLLÁSÚ eléréséhez használt. Az Azure Load Balancer példányok az alkalmazás réteg alhálózatban lévő virtuális gépek felé irányuló forgalom terjesztésére szolgálnak.

**Rendelkezésre állási csoportok**. Virtuális gépek a SAP webes kézbesítő, SAP alkalmazáskiszolgáló és (A) SCS, szerepköröket külön vannak csoportosítva [rendelkezésre állási készletek](/azure/virtual-machines/windows/tutorial-availability-sets), és legalább két virtuális gép szerepkör / törlődnek. Ez lehetővé teszi a virtuális gépeket abban az esetben jogosult a magasabb [szolgáltatói szerződést](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA).

**Hálózati adapter**. [A hálózati kártyák](/azure/virtual-network/virtual-network-network-interface) (NIC) a virtuális hálózat virtuális gépei az összes kommunikáció engedélyezése.

**Hálózati biztonsági csoportok**. Bejövő korlátozásához kimenő, és az alhálózatok közötti helyen belüli forgalmat a virtuális hálózat hozhat létre [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-k).

**Átjáró**. Átjáró kiterjeszti a helyszíni hálózat az Azure virtuális hálózat. [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) az ajánlott Azure szolgáltatás, amely ne lépje át a nyilvános internethez magánhálózati kapcsolatok létrehozására, de egy [pont-pont](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) kapcsolat is.

**Azure Storage**. A virtuális gép virtuális merevlemez (VHD), állandó tárolást [Azure Storage](/azure/storage/storage-standard-storage) szükséges. Azt is használják [felhő tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) egy feladatátvételi fürt művelet végrehajtásához. 

## <a name="recommendations"></a>Javaslatok
Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="sap-web-dispatcher-pool"></a>SAP webes kézbesítő készlet

A webes kézbesítő összetevő SAP forgalom között az SAP alkalmazáskiszolgálók terheléselosztó használható. A webes kézbesítő összetevő magas rendelkezésre állásának eléréséhez, az Azure Load Balancer a párhuzamos webes kézbesítő telepítés végrehajtásához használatos. Webes kézbesítő ciklikus multiplexelés konfigurációban használ HTTP (S)-forgalom elosztását a rendelkezésre álló webes elosztás a terheléselosztó-készlet.

Az Azure virtuális gépeken SAP NetWeaver futtatásával kapcsolatos részletekért lásd: [Azure virtuális gépek tervezési és megvalósítási az SAP NetWeaver](/azure/virtual-machines/workloads/sap/planning-guide).

### <a name="application-servers-pool"></a>Alkalmazáskészlet-kiszolgálók

Bejelentkezési csoportok ABAP alkalmazáskiszolgálók kezeléséhez, a SMLG tranzakció szolgál. A terheléselosztási függvény központi szolgáltatások üzenet-kiszolgálón belül használja SAPGUIs és RFC oszthatja meg a munkaterhelést SAP alkalmazáskészlet-kiszolgálók közötti forgalmat. Az alkalmazás server kapcsolat a magas rendelkezésre állású központi szolgáltatások nem keresztül a fürt virtuális hálózat neve.

### <a name="sap-central-services-cluster"></a>SAP szolgáltatásokhoz központi fürtre

A referencia-architektúrában központi szolgáltatások virtuális gépeken futtatja az alkalmazáshoz tartozó. A központi szolgáltatások egy potenciális hibaérzékeny pontot (SPOF) egy virtuális központi telepítési – tipikus telepítése esetén magas rendelkezésre állás nem követelmény. Egy magas rendelkezésre állású megoldás bevezetésére, vagy egy megosztott lemezfürt, vagy egy fájlkiszolgáló-megosztás fürt használható.

Virtuális gépek megosztott lemezfürt konfigurálásához használja [Windows Server feladatátvevő fürt](https://blogs.sap.com/2018/01/25/how-to-create-sap-resources-in-windows-failover-cluster/). [A felhő tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) egy kvórum tanúsító ajánlott. A feladatátvevő fürt környezet támogatásához [SIOS DataKeeper Cluster Edition](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8) által a fürtcsomópontok által birtokolt független lemezek replikálásához a fürt megosztott kötet funkciót hajt végre. Azure natív módon nem támogatja a megosztott lemezeket, és ezért a SIOS által biztosított megoldások szükségesek.

További információkért lásd: "3. Fontos frissítést az SAP ügyfelek futtató ASC Azure SIOS a" [SAP futó alkalmazások a Microsoft platformon](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/).

Fürtszolgáltatás kezelésére úgy, hogy a fájl megosztási fürt Windows Server feladatátvevő fürt alkalmazza. [SAP](https://blogs.sap.com/2018/03/19/migration-from-a-shared-disk-cluster-to-a-file-share-cluster/) nemrég módosította a szolgáltatásokhoz központi telepítési minta a /sapmnt globális könyvtárak keresztül UNC elérési út elérésére. Ez a változás [nincs szükség](https://blogs.msdn.microsoft.com/saponsqlserver/2017/08/10/high-available-ascs-for-windows-on-file-share-shared-disk-no-longer-required/) SIOS vagy más megosztott lemez megoldások a központi szolgáltatások virtuális gépeken. Győződjön meg arról, hogy van-e a /sapmnt UNC-megosztásnevet továbbra is ajánlott [magas rendelkezésre állású](https://blogs.sap.com/2017/07/21/how-to-create-a-high-available-sapmnt-share/). Ehhez a központi Services-példány által a Windows Server feladatátvevő fürt használatával [kibővített fájlkiszolgáló](https://blogs.msdn.microsoft.com/saponsqlserver/2017/11/14/file-server-with-sofs-and-s2d-as-an-alternative-to-cluster-shared-disk-for-clustering-of-an-sap-ascs-instance-in-azure-is-generally-available/) (SOFS) és a [közvetlen tárolóhelyek](https://blogs.sap.com/2018/03/07/your-sap-on-azure-part-5-ascs-high-availability-with-storage-spaces-direct/) (S2D) funkció a Windows Server 2016. 

### <a name="availability-sets"></a>Rendelkezésre állási csoportok

Rendelkezésre állási készletek terjesztése a kiszolgálókat különböző fizikai infrastruktúra és a frissítési csoport szolgáltatás rendelkezésre állásának javítása érdekében. Helyezze olyan virtuális gépek, amelyek ugyanarra a szerepkörre végre a rendelkezésre állási beállítja a megfelelő és segítenek védelmet biztosítanak a Azure-infrastruktúra karbantartás miatt [SLA-k](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA). Ajánlott legalább két virtuális gép rendelkezésre állási csoportban.

Csoportban lévő összes virtuális gépet ugyanarra a szerepkörre kell végrehajtania. Ne keverje a kiszolgálókat különböző szerepkörök az azonos rendelkezésre állási készlet. Például nem helyezhető el egy központi szolgáltatások csomópont azonos rendelkezésre állási csoportban, az alkalmazás-kiszolgálóval.

### <a name="nics"></a>Hálózati adapterek (NIC-k)

Hagyományos helyszíni SAP központi telepítések több hálózati adapterek (NIC) elkülönítse a felügyeleti forgalom és a business forgalom gépenként valósítja meg. Az Azure a virtuális hálózat a szoftver által meghatározott hálózati által küldött összes forgalom az azonos hálózati háló keresztül. Ezért a több hálózati adapter használata szükségtelen. Azonban ha a szervezetének korlátoznia kell áthaladó forgalmat leválasszanak, telepítheti több hálózati adapter virtuális gépenként, minden egyes hálózati adapter csatlakoztatása egy másik alhálózatot, és az NSG-k segítségével különböző hozzáférés-vezérlési házirendek kényszerítése.

### <a name="subnets-and-nsgs"></a>Alhálózatokat és az NSG-k

Ez az architektúra a virtuális hálózat címtere alterületét alhálózatokra. A referencia-architektúrában elsősorban az alkalmazás réteg alhálózat összpontosít. Az egyes alhálózatokon társítható egy NSG-t, amely meghatározza a hozzáférési házirendek az alhálózat. Alkalmazáskiszolgálók helyezze külön alhálózathoz, biztonságát azokat könnyebben alhálózati biztonsági házirendeket, nem az egyes kiszolgálók kezeléséhez.

Amikor egy NSG-t hozzárendelnek egy alhálózathoz, akkor az alhálózaton belüli megadott kiszolgálókra vonatkoznak. A minden részletre kiterjedő szabályozhatják a kiszolgálók az alhálózat NSG-k használatával kapcsolatos további információkért lásd: [hálózati forgalmat hálózati biztonsági csoportokkal](https://azure.microsoft.com/en-us/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/).

### <a name="load-balancers"></a>Terheléselosztók

[SAP webes kézbesítő](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) leírók terheléselosztása HTTP (S) forgalmat egy SAP alkalmazáskiszolgálók készletéhez.

A Diagnosztika protokollt vagy a távoli függvény hívások (RFC) keresztül egy SAP-kiszolgálóhoz csatlakozó SAP GUI-ügyfelektől érkező forgalom, a központi szolgáltatások üzenet kiszolgáló osztja szét SAP alkalmazás kiszolgálón keresztül [csoportok bejelentkezési](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing), ezért nem extra terhelést terheléselosztó van szükség.

### <a name="azure-storage"></a>Azure Storage

Az összes adatbázis kiszolgáló virtuális gép azt javasoljuk, prémium szintű Azure Storage következetes olvasás/írás késésének. Prémium szintű Storage használata az összes operációs rendszer lemez és adatlemezek egypéldányos virtuális gépi, lásd: [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines). Emellett éles rendszerek esetén SAP, azt javasoljuk, prémium szintű [Azure felügyelt lemezek](/azure/storage/storage-managed-disks-overview) minden esetben. A megbízhatóság kezelt lemezek segítségével kezelheti a VHD-fájlokat, a lemezek. Kezelt lemezek biztosítsa, hogy a kötet a virtuális gépek rendelkezésre állási csoportok belül elkülönített elkerülése érdekében a hibaérzékeny pontokat.

Az SAP alkalmazáskiszolgálók, többek között a központi szolgáltatások virtuális gépeket a Azure standard szintű tárolást segítségével csökkenthető a költség, mert alkalmazásfuttatás történik, memória és a lemezek használt csak naplózás. Azonban ekkor, Standard szintű tárolót csak minősítéssel nem felügyelt tárolási. Alkalmazáskiszolgálók tárol adatokat, mert is használhatják a kisebb P4 és P6 prémium szintű Storage lemezeket költségek csökkentése érdekében.

Az Azure Storage is használják [felhő tanúsító](/windows-server/failover-clustering/deploy-cloud-witness) kvórum fenntartása az eszközök a távoli Azure-régió, az elsődleges régióban, amelyben a fürtben található másik lapra.

A biztonsági mentési tárolót, ajánljuk Azure [coolaccess réteg](/azure/storage/storage-blob-storage-tiers) és [hozzáférési réteg archivált](/azure/storage/storage-blob-storage-tiers). A tárolási rétegek költséghatékony módon hosszú élettartamú, amelyek ritkán használt adatok tárolására.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Az adatbázis-kiszolgálók állandó kommunikációjához SAP alkalmazáskiszolgálók végzi. A teljesítmény szempontjából kulcsfontosságú alkalmazásokkal futó minden adatbázis-platformokon, beleértve az SAP HANA-érdemes engedélyezni [írási gyorsító](/azure/virtual-machines/linux/how-to-enable-write-accelerator) napló írási késése javítása érdekében. A helyek közötti kiszolgálói kommunikáció optimalizálása érdekében használja a [az elérését gyorsítja fel hálózati](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Ne feledje, hogy ezek a gyorsbillentyűk csak bizonyos Virtuálisgép-sorozat érhető el.

A gyakori eljárásokat magas iops-érték és a lemez sávszélesség átviteli sebesség eléréséhez a tárolókötetet [teljesítményoptimalizálás](/azure/virtual-machines/windows/premium-storage-performance) Azure tárolási elrendezés vonatkozik. Például több lemezek csoportosításával csíkozott kötet létrehozása kombinálásával növeli IO teljesítményét. A tárolóban található ritkán változó tartalmak olvasási gyorsítótárazásának engedélyezésével növelhető az adatelérés sebessége.

Az SAP SQL a [felső 10 kulcs szempontjai SAP-alkalmazások központi telepítése az Azure-on](https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/25/top-10-key-considerations-for-deploying-sap-applications-on-azure/) blog kiváló tanácsokat az SAP-munkaterhelések SQL-kiszolgálón az Azure storage optimalizálásához.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az SAP-alkalmazás rétegben az Azure számos különböző vertikális felskálázásával és kiterjesztése a virtuálisgép-méretek kínál. A két szélsőértéket beleértve listája, [SAP Megjegyzés 1928533](https://launchpad.support.sap.com/#/notes/1928533) -Azure SAP-alkalmazásokból: támogatott termékek és Azure Virtuálisgép-típusokon. (SAP szolgáltatás piactér fiók-hozzáférés szükséges.) SAP alkalmazás-kiszolgálókat és a központi szolgáltatások fürtök méretezheti fel/le vagy terjessze ki több példány hozzáadásával. A AnyDB adatbázis méretezheti fel/le, de nem kiterjesztése. Az SAP adatbázis tárolója AnyDB nem támogatja a horizontális.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás redundanciát a magas rendelkezésre állású infrastruktúramegoldásainak általános téma. A vállalatok számára, amelyek kevésbé szigorú SLA-t egypéldányos Azure virtuális gépek nyújtanak hasznos üzemidőt SLA-t. További információkért lásd: [Azure szolgáltatásiszint-megállapodás](https://azure.microsoft.com/support/legal/sla/).

Ez az SAP-alkalmazás elosztott telepítés, a alaptelepítésének replikálódik, magas rendelkezésre állás biztosítása érdekében. Az egyes rétegekhez a architektúra a magas rendelkezésre állású kialakítása függően változik.

### <a name="application-tier"></a>Alkalmazás réteg

Magas rendelkezésre állás a SAP webes kézbesítő redundáns osztályt érhető el. Lásd: [SAP webes kézbesítő](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) az SAP-dokumentációban.

Magas rendelkezésre állású a központi szolgáltatásokat a Windows Server feladatátvevő fürt segítségével valósul meg. Az Azure-on történő telepítésekor a fürttároló, a feladatátvevő fürt használatával konfigurálható két megközelítés: vagy a fürtözött megosztott kötet vagy egy fájlmegosztáson.

Mivel a megosztott lemezeket nem lehetséges az Azure-on, SIOS Datakeeper a tartalom a fürtcsomópontokhoz kapcsolt független lemezek replikálásához használt, és a meghajtók absztrakt fürtként megosztott kötet a fürt kezelő. Megvalósítás részletei: [SAP ASC fürtszolgáltatás Azure](https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/).

Egy másik lehetőség egy által kiszolgált egy fájlmegosztást a [Scale-Out fájlkiszolgáló](https://blogs.msdn.microsoft.com/saponsqlserver/2017/11/14/file-server-with-sofs-and-s2d-as-an-alternative-to-cluster-shared-disk-for-clustering-of-an-sap-ascs-instance-in-azure-is-generally-available/) (SOFS). Kibővíthető Fájlkiszolgáló ajánlatokat a refs fájlrendszer megosztások is használhatja a fürt megosztott kötetéhez a Windows-fürt. A kibővíthető Fájlkiszolgáló fürt több központi szolgáltatások csomópontok között megosztható legyen. Től írásának pillanatában kibővíthető Fájlkiszolgáló használata csak a magas rendelkezésre állású kialakítása, a kibővíthető Fájlkiszolgáló fürt nem bővíti ki a vész-helyreállítási támogatás biztosítható régiók között.

Magas rendelkezésre állásúvá tenni a SAP alkalmazáskiszolgálók terheléselosztási forgalmat alkalmazáskiszolgálók készletét belül az érhető el.
Lásd: [SAP tanúsítványokról és a Microsoft Azure-on futó konfigurációk](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="database-tier"></a>Adatbázis-rétegből

A referencia-architektúrában azt feltételezi, hogy a forráshely adatbázisára AnyDB futó – Ez azt jelenti, hogy egy adatbázis-kezelő például az SQL Server, SAP ASE, IBM DB2 vagy Oracle. Az adatbázis-rétegből natív replikációs szolgáltatása replikált csomópontok közötti kézi vagy automatikus feladatátvételt.

Megvalósítás részletei adott adatbázis-rendszerekkel kapcsolatban, lásd: [SAP NetWeaver Azure virtuális gépek DBMS telepítésének](/azure/virtual-machines/workloads/sap/dbms-guide).

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

A vészhelyreállítás (DR) egy másodlagos régióba történő feladatátvételt képesnek kell lennie. Minden egyes réteg más stratégiával biztosít vészhelyreállítási (DR) védelmet.

- **Alkalmazás-kiszolgálók réteg**. SAP alkalmazáskiszolgálók nem tartalmaz üzleti adatokat. A Azure-ban egy egyszerű vész-Helyreállítási stratégia, ha a SAP alkalmazáskiszolgálók másodlagos régióban, állítsa le. Bármilyen konfigurációs módosításokat vagy az elsődleges alkalmazáskiszolgálón kernel frissítések esetén módosítások kell másolni a virtuális gépek szerepelnek a másodlagos régióba. Például a kernel végrehajtható fájlokat másolni a vész-Helyreállítási virtuális gépek. Automatikus replikációja egy másodlagos régióban alkalmazáskiszolgálók, [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) a javasolt megoldás.

- **Központi szolgáltatások**. Ez az összetevő a SAP alkalmazás verem is nem maradnak üzleti adatok. Létrehozhat egy virtuális Gépet a vész-helyreállítási régióban a központi szolgáltatások szerepkör futtatásához. Csak a tartalmi szinkronizálni az elsődleges központi szolgáltatások csomópontból-e a /sapmnt megosztást. Is ha a konfigurációs módosítások és a kernel frissítéseket az elsődleges kiszolgálón központi szolgáltatások történik, azok meg kell ismételni a virtuális gép központi szolgáltatásokat futtató vész-helyreállítási régióban. A két kiszolgáló szinkronizálásához, vagy Azure Site Recovery segítségével replikálni a fürtcsomópontokon, vagy egyszerűen csak egy rendszeres, ütemezett feladat segítségével /sapmnt másolja a vész-helyreállítási régió. Ez egyszerű módszert build, a másolási és a teszt feladatátvételi folyamat, a letöltés vonatkozó további információért [SAP NetWeaver: egy Hyper-V és a Microsoft Azure-alapú vész-helyreállítási megoldást felépítése](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx), és tekintse meg a "4.3. SAP SPOF layer (ASCS)” (SAP SPOF réteg – ASCS) fejezetet.

- **Adatbázis-rétegből**. DR legjobban az adatbázis saját integrált replikációs technológiával hajtanak végre. SQL Server esetén például azt javasoljuk, AlwaysOn rendelkezésre állási csoport létrehozni egy távoli régióban replika kézi feladatátvételre tranzakciót aszinkron módon replikál. Aszinkron replikáció elkerülhető az hatással lehet a teljesítmény interaktív munkaterhelések az elsődleges helyen. Kézi feladatátvételre lehetővé teszi, hogy az a személy vész-Helyreállítási hatásának vizsgálatában, és döntse el, ha a vész-Helyreállítási helyről működő indokolt.

Azure Site Recovery segítségével automatikusan készítse el egy teljes mértékben replikált munkakörnyezeti helyet az eredeti, futtatnia kell testre szabott [üzembe helyezési parancsfájlok](/azure/site-recovery/site-recovery-runbook-automation). A Site Recovery először telepíti a virtuális gépek rendelkezésre állási csoportok, majd futtat parancsfájlokat, mint az erőforrások hozzáadása a terheléselosztók.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Azure biztosít több függvények [figyelése és diagnosztikai](/azure/architecture/best-practices/monitoring) az általános infrastruktúra. Emellett az Azure virtuális gépek fokozott ellenőrzés kezeli Azure Operations Management Suite (OMS).

SAP alapú felügyelete, erőforrások és a szolgáltatás teljesítménye az SAP infrastruktúra biztosításához a [Azure SAP fokozott ellenőrzésére](/azure/virtual-machines/workloads/sap/deployment-guide#detailed-tasks-for-sap-software-deployment) kiterjesztését. Ez a bővítmény betölti az Azure monitorozási statisztikáit az SAP alkalmazásba az operációs rendszer monitorozása és a DBA Cockpit funkcióinak használata céljából.

## <a name="security-considerations"></a>Biztonsági szempontok

Az SAP egy saját felhasználófelügyeleti motorral (UME-vel) vezérli a szerepköralapú hozzáféréseket és az engedélyezést az SAP alkalmazáson belül. További információkért lásd: [SAP NetWeaver alkalmazáskiszolgáló ABAP biztonsági útmutató](https://help.sap.com/doc/7b932ef4728810148a4b1a83b0e91070/1610 001/en-US/frameset.htm?4dde53b3e9142e51e10000000a42189c.html) és [SAP NetWeaver Server Java biztonsági útmutatója](https://help.sap.com/doc/saphelp_snc_uiaddon_10/1.0/en-US/57/d8bfcf38f66f48b95ce1f52b3f5184/frameset.htm).

A további hálózati biztonság érdekében vegye fontolóra egy [DMZ hálózati](../dmz/secure-vnet-hybrid.md), a hálózati virtuális készülék használó webes kézbesítő a tűzfal elé az alhálózat létrehozásához.

Biztonsági infrastruktúra az adatok titkosítása az átvitel során, és inaktív. A "Biztonsági szempontok" szakaszában a [SAP NetWeaver az Azure virtuális gépek (VM) – tervezési és megvalósítási útmutató](/azure/virtual-machines/workloads/sap/planning-guide) történő címet hálózati biztonsági kezdődik. Ez az útmutató adja meg azt is, hogy a tűzfalakon mely portokat kell megnyitni az alkalmazás kommunikációjának lehetővé tételéhez.

Windows virtuális gépek lemezeit titkosításához, használhatja a [Azure Disk Encryption](/azure/security/azure-security-disk-encryption). A BitLocker Windows-szolgáltatás segítségével kötettitkosítást biztosít az operációs rendszer és az adatlemezek. A megoldás is használható az Azure Key Vault segítségével szabályozhatja, és a lemez-titkosítási kulcsok és titkos kulcsainak kulcstároló-előfizetése kezeléséhez. A virtuális gépek lemezein található, inaktív adatok az Azure Storage-ban titkosítva vannak.

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

- [A Microsoft Platform blog futó SAP-alkalmazásokból](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Az Azure közösségi támogatás](https://azure.microsoft.com/support/community/)
- [SAP-Közösség](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)

