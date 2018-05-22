---
title: SAP S/4HANA Azure Linux virtuális gépekhez
description: Eljárások az SAP S/4HANA környezetben futó Linux Azure magas rendelkezésre állású bizonyítása.
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: d24ef6f9e4eae460d0d0dcfff35568c812d09951
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/21/2018
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>SAP S/4HANA Azure Linux virtuális gépekhez

A referencia-architektúrában az S/4HANA egy magas rendelkezésre állású környezet, amely támogatja a vész-helyreállítási az Azure-on futó bevált gyakorlatok csoportja jeleníti meg. Ez az architektúra van szükség az adott virtuális gép (VM), amelyek módosíthatók a szervezet igényeinek megfelelően van telepítve. 


![](./images/sap-s4hana.png)

## <a name="architecture"></a>Architektúra
 
> [!NOTE] 
> A referencia-architektúrában megfelelően SAP termékek alkalmazástelepítéshez szükséges megfelelő licencelési SAP-termékek és más nem Microsoft-technológiák.

A referencia-architektúrában ismerteti a vállalati szintű, éles szintű rendszeren. Az üzleti kell, ez a konfiguráció egyetlen virtuális gép lehet korlátozni. Azonban a következő összetevők szükségesek:

**Virtuális hálózat**. A [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) szolgáltatás biztonságosan csatlakozik Azure-erőforrások egymással. Ebben a rendszerben a virtuális hálózat egy helyszíni környezetben telepített központban az átjárón keresztül csatlakozik egy [hub-küllős topológia](../hybrid-networking/hub-spoke.md). A küllős a virtuális hálózat az SAP-alkalmazásokhoz.

**Alhálózatok**. A virtuális hálózathoz van felosztva, külön [alhálózatok](/azure/virtual-network/virtual-network-manage-subnet) az egyes rétegekhez: átjáró, az alkalmazás, az adatbázis és a megosztott szolgáltatások. 

**Virtuális gépek**. Ez az architektúra használja az alkalmazás réteg és adatbázis-rétegből, az alábbiak szerint csoportosítva Linuxot futtató virtuális gépek:

- **Alkalmazás réteg**. Tartalmazza a Fiori előfeldolgozó kiszolgálókészlethez, SAP webes kézbesítő készlet, alkalmazáskészletet kiszolgáló, és a központi SAP-szolgáltatások fürt. Magas rendelkezésre állású központi szolgáltatásokat Azure Linux virtuális gépeken meg kell adni egy magas rendelkezésre állású hálózati fájlrendszer (NFS) szolgáltatás.
- **NFS-fürt**. Ez az architektúra használ egy [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) server rendelkező Linux fürtökön SAP-rendszerek között megosztott adatok tárolására. A központi fürtre is oszthatók meg több SAP rendszer. Az NFS-szolgáltatás magas rendelkezésre állása a megfelelő magas rendelkezésre állási a kijelölt Linux terjesztési kiterjesztését.
- **SAP HANA**. Az adatbázis-rétegből két vagy több Linux virtuális gépek a fürt magas rendelkezésre állás biztosítása érdekében használja. HANA rendszer replikációs (HSR) segítségével tartalma replikálja az elsődleges és másodlagos HANA rendszerek között. Linux-Fürtszolgáltatás észleli a rendszer hibáit, és automatikus feladatátvételt elősegítésére szolgál. Egy olyan tároló vagy felhő alapú kerítés mechanizmus segítségével a sikertelen rendszer elkülönített, vagy állítsa le a fürt maszkolt feltétel elkerülése érdekében győződjön meg arról.
- **Jumpbox**. Más néven bástyagazdagép. Ez a biztonságos virtuális gép, amely a rendszergazdák használhatják a virtuális számítógépekhez kapcsolódni a hálózaton. A Windows vagy Linux futtatható. Használjon egy Windows jumpbox webböngésző kényelmi HANA vezérlőpultot vagy HANA Studio eszközök használata esetén.

**Terheléselosztók**. Mindkét SAP beépített terheléselosztók és [Azure terheléselosztó](/azure/load-balancer/load-balancer-overview) magas rendelkezésre ÁLLÁSÚ eléréséhez használt. Az Azure Load Balancer példányok az alkalmazás réteg alhálózatban lévő virtuális gépek felé irányuló forgalom terjesztésére szolgálnak.

**Rendelkezésre állási csoportok**. A készletek és a fürtök (Web kézbesítő, SAP alkalmazáskiszolgálók, központi szolgáltatások, NFS és HANA) virtuális gépek vannak csoportosítva, különálló [rendelkezésre állási készletek](/azure/virtual-machines/windows/tutorial-availability-sets), és legalább két virtuális gép szerepkör / törlődnek. Ez lehetővé teszi a virtuális gépeket abban az esetben jogosult a magasabb [szolgáltatói szerződést](https://azure.microsoft.com/support/legal/sla/virtual-machines) (SLA). 

**Hálózati adapter**. [A hálózati kártyák](/azure/virtual-network/virtual-network-network-interface) (NIC) a virtuális hálózat virtuális gépei az összes kommunikáció engedélyezése.

**Hálózati biztonsági csoportok**. Bejövő korlátozásához kimenő, és az alhálózatok közötti helyen belüli forgalmat a virtuális hálózat [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-k) használata.

**Átjáró**. Átjáró kiterjeszti a helyszíni hálózat az Azure virtuális hálózat. [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute) az ajánlott Azure szolgáltatás, amely ne lépje át a nyilvános internethez magánhálózati kapcsolatok létrehozására, de egy [pont-pont](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) kapcsolat is. 

**Azure Storage**. A virtuális gép virtuális merevlemez (VHD), állandó tárolást [Azure Storage](/azure/storage/) szükséges. 

## <a name="recommendations"></a>Javaslatok

Ez az architektúra ismerteti egy kis termelési szint vállalati üzembe helyezéséhez. Eltérőek, hogy a központi telepítés üzleti igényei alapján. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="virtual-machines"></a>Virtual machines (Virtuális gépek)

Alkalmazáskészletek kiszolgáló és a fürtök állítsa be úgy a követelmények alapján virtuális gépek száma. A [Azure virtuális gépek tervezési és megvalósítási útmutató](/azure/virtual-machines/workloads/sap/planning-guide) tartalmazza a virtuális gépek hamarosan futó SAP NetWeaver, de az információk SAP S/4HANA is vonatkozik.

SAP adatait támogatja az Azure virtuális gépek típusainak és átviteli sebességre vonatkozó mérőszámok (SAP), a következő témakörben: [SAP Megjegyzés 1928533](https://launchpad.support.sap.com/#/notes/1928533). 

### <a name="sap-web-dispatcher-pool"></a>SAP webes kézbesítő készlet

A webes kézbesítő összetevő SAP forgalom között az SAP alkalmazáskiszolgálók terheléselosztó használható. A webes kézbesítő összetevő magas rendelkezésre állásának eléréséhez, Azure Load Balancer szolgál a párhuzamos webes kézbesítő telepítő megvalósítását a ciklikus multiplexelési konfigurációjának a rendelkezésre álló webes elosztás a terheléselosztók a forgalom eloszlása HTTP (S) háttér-készlet. 

### <a name="fiori-front-end-server"></a>Fiori előtér-kiszolgáló

A Fiori előtér-kiszolgáló használ a [NetWeaver átjáró](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true). Kisebb környezetekben is betölthető a Fiori kiszolgálón. A nagyméretű telepítések esetében a NetWeaver átjáró külön kiszolgáló lehet, hogy telepítette a Fiori előfeldolgozó kiszolgálókészlet elé.

### <a name="application-servers-pool"></a>Alkalmazáskészlet-kiszolgálók

Bejelentkezési csoportok ABAP alkalmazáskiszolgálók kezeléséhez, a SMLG tranzakció szolgál. A terheléselosztási függvény központi szolgáltatások üzenet-kiszolgálón belül használja SAPGUIs és RFC oszthatja meg a munkaterhelést SAP alkalmazáskészlet-kiszolgálók közötti forgalmat. Az alkalmazás server kapcsolat a magas rendelkezésre állású központi szolgáltatások nem keresztül a fürt virtuális hálózat neve. Ezzel elkerülhető, hogy a helyi feladatátvétele után az alkalmazás server-profil központi szolgáltatások csatlakozási megváltoztatását. 

### <a name="sap-central-services-cluster"></a>SAP szolgáltatásokhoz központi fürtre

Központi szolgáltatások egyetlen virtuális gépre telepíthető, ha magas rendelkezésre állás nem követelmény. Azonban az egyetlen virtuális gép lesz a potenciális hibaérzékeny pontok kialakulását (SPOF) az SAP-környezet. Egy magas rendelkezésre állású szolgáltatások központi telepítése esetén az NFS magas rendelkezésre állású fürt- és egy központi szolgáltatások magas rendelkezésre állású fürt használt.

### <a name="nfs-cluster"></a>NFS-fürt

Az NFS-fürt csomópontjai közötti replikáció használható DRBD (elosztott replikált elérésű eszközt használ).

### <a name="availability-sets"></a>Rendelkezésre állási csoportok

Rendelkezésre állási készletek terjesztése a kiszolgálókat különböző fizikai infrastruktúra és a frissítési csoport szolgáltatás rendelkezésre állásának javítása érdekében. Helyezze olyan virtuális gépek, amelyek ugyanarra a szerepkörre végre a rendelkezésre állási beállítja a megfelelő és segítenek védelmet biztosítanak a Azure-infrastruktúra karbantartás miatt [SLA-k](https://azure.microsoft.com/support/legal/sla/virtual-machines). Ajánlott legalább két virtuális gép rendelkezésre állási csoportban.

Csoportban lévő összes virtuális gépet ugyanarra a szerepkörre kell végrehajtania. Ne keverje a kiszolgálókat különböző szerepkörök az azonos rendelkezésre állási készlet. Például ne tegyen ASC csomópont azonos rendelkezésre állási csoportban, az alkalmazás-kiszolgálóval.

### <a name="nics"></a>Hálózati adapterek (NIC-k)

Hagyományos helyszíni SAP tájak több hálózati adapterek (NIC) elkülönítse a felügyeleti forgalom és a business forgalom gépenként valósítja meg. Az Azure a virtuális hálózat a szoftver által meghatározott hálózati által küldött összes forgalom az azonos hálózati háló keresztül. Ezért a több hálózati adapter használata szükségtelen. Azonban ha a szervezetének korlátoznia kell áthaladó forgalmat leválasszanak, telepítheti több hálózati adapter virtuális gépenként, minden egyes hálózati adapter csatlakoztatása egy másik alhálózatot, és az NSG-k segítségével különböző hozzáférés-vezérlési házirendek kényszerítése.

### <a name="subnets-and-nsgs"></a>Alhálózatokat és az NSG-k

Ez az architektúra a virtuális hálózat címtere alterületét alhálózatokra. Az egyes alhálózatokon társítható egy NSG-t, amely meghatározza a hozzáférési házirendek az alhálózat. Alkalmazáskiszolgálók helyezze külön alhálózathoz, biztonságát azokat könnyebben alhálózati biztonsági házirendeket, nem az egyes kiszolgálók kezeléséhez.

Amikor egy NSG-t hozzárendelnek egy alhálózathoz, majd vonatkozik, az alhálózaton lévő összes kiszolgálót. A minden részletre kiterjedő szabályozhatják a kiszolgálók az alhálózat NSG-k használatával kapcsolatos további információkért lásd: [hálózati forgalmat hálózati biztonsági csoportokkal](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/).

Lásd még: [tervezése és kialakítása VPN-átjáró](/azure/vpn-gateway/vpn-gateway-plan-design).

### <a name="load-balancers"></a>Terheléselosztók

[SAP webes kézbesítő](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm) leírók terheléselosztása Fiori stílus alkalmazást egy SAP alkalmazáskiszolgáló HTTP (S) forgalmat. 

A Diagnosztika vagy függvény hívások (RFC) keresztül egy SAP-kiszolgálóhoz csatlakozó SAP GUI-ügyfelektől érkező forgalom, a központi szolgáltatás üzenet kiszolgáló osztja szét SAP alkalmazás kiszolgálón keresztül [csoportok bejelentkezési](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing), így a nem a további terheléselosztó szükséges. 

### <a name="azure-storage"></a>Azure Storage

Prémium szintű Azure Storage az adatbázis-kiszolgáló virtuális gépek használatát javasoljuk. Prémium szintű storage következetes olvasás/írás késést biztosít. Prémium szintű Storage segítségével az operációs rendszer és a adatok lemezek egypéldányos a virtuális gép kapcsolatos részletekért lásd: [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

Minden éles SAP rendszerek esetén, azt javasoljuk, prémium szintű [Azure felügyelt lemezek](/azure/storage/storage-managed-disks-overview). Felügyelt lemezek segítségével kezelheti a VHD-fájlokat, a lemezek megbízhatóság hozzáadása. Azt is biztosítják, hogy a lemezek, a virtuális gépek rendelkezésre állási csoportok belül választva egymástól a hibaérzékeny pontokat elkerülése érdekében.

Az SAP alkalmazáskiszolgálók, többek között a központi szolgáltatások virtuális gépeket a Azure standard szintű tárolást segítségével csökkenthető a költség, mert alkalmazásfuttatás történik, memória, és használja a lemezek csak naplózás. Azonban ekkor, Standard szintű tárolót csak minősítéssel nem felügyelt tárolási. Alkalmazáskiszolgálók tárol adatokat, mert is használhatják a kisebb P4 és P6 prémium szintű Storage lemezeket költségek csökkentése érdekében.

A biztonsági mentési tárolót, ajánljuk Azure [cool réteg tároló elérése és/vagy a hozzáférési réteg archivált](/azure/storage/storage-blob-storage-tiers). A tárolási rétegek költséghatékony módon, hogy az ritkábban hosszú élettartamú adatok tárolására.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Az adatbázis-kiszolgálók állandó kommunikációjához SAP alkalmazáskiszolgálók végzi. A HANA adatbázis virtuális gépekhez, érdemes lehet engedélyezni az [írási gyorsító](/azure/virtual-machines/linux/how-to-enable-write-accelerator) napló írási késése javítása érdekében. A helyek közötti kiszolgálói kommunikáció optimalizálása érdekében használja a [az elérését gyorsítja fel hálózati](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/). Ne feledje, hogy ezek a gyorsbillentyűk csak bizonyos Virtuálisgép-sorozat érhető el.

A gyakori eljárásokat magas iops-érték és a lemez sávszélesség átviteli sebesség eléréséhez a tárolókötetet [teljesítményoptimalizálás](/azure/virtual-machines/linux/premium-storage-performance) Azure tárolási elrendezés vonatkozik. Például több lemezek csoportosításával csíkozott kötet létrehozása kombinálásával növeli IO teljesítményét. A tárolóban található ritkán változó tartalmak olvasási gyorsítótárazásának engedélyezésével növelhető az adatelérés sebessége. Teljesítmény-követelményekkel kapcsolatos részletekért lásd: [SAP Megjegyzés 1943937 - hardver konfiguráció ellenőrzése eszköz](https://launchpad.support.sap.com/#/notes/1943937) (SAP szolgáltatás piactér a hozzáféréshez szükséges fiók).

## <a name="scalability-considerations"></a>Méretezési szempontok

Az SAP-alkalmazás rétegben az Azure számos különböző vertikális felskálázásával és kiterjesztése a virtuálisgép-méretek kínál. A két szélsőértéket beleértve listája, [SAP Megjegyzés 1928533](https://launchpad.support.sap.com/#/notes/1928533) -Azure SAP-alkalmazásokból: támogatott termékek és az Azure virtuális gép típusok (SAP szolgáltatás piactér a hozzáféréshez szükséges fiók). Több virtuális gépek típushoz tanúsítására folyamatos, méretezhető felfelé vagy lefelé az azonos felhőben üzemelő példányhoz. 

Az adatbázisrétegben az ebbe az architektúrába HANA futó virtuális gépek. Ha a terhelést meghaladja a maximális Virtuálisgép-méretet, a Microsoft is kínál [Azure nagy példányok](/azure/virtual-machines/workloads/sap/hana-overview-architecture) SAP Hana. Ezek a fizikai kiszolgálók egy helyen találhatók a Microsoft Azure datacenter hitelesített, és frissítésétől írásának, adja meg legfeljebb 20 TB memória kapacitást biztosít egy példányban. Több csomópont-konfiguráció esetében is lehetséges a teljes memória 60 TB kapacitású.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás redundanciát a magas rendelkezésre állású infrastruktúramegoldásainak általános téma. A vállalatok számára, amelyek kevésbé szigorú SLA-t egypéldányos Azure virtuális gépek nyújtanak hasznos üzemidőt SLA-t. További információkért lásd: [Azure szolgáltatásiszint-megállapodás](https://azure.microsoft.com/support/legal/sla/).

Ez az SAP-alkalmazás elosztott telepítés, a alaptelepítésének replikálódik, magas rendelkezésre állás biztosítása érdekében. Az egyes rétegekhez a architektúra a magas rendelkezésre állású kialakítása függően változik. 

### <a name="application-tier"></a>Alkalmazás réteg

- Webes kézbesítő. Magas rendelkezésre állású redundáns webes kézbesítő osztályt érhető el. Lásd: [SAP webes kézbesítő](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm) az SAP-dokumentációban.
- Fiori kiszolgálók. Magas rendelkezésre állás az terheléselosztási forgalmat egy kiszolgálókészletet belül érhető el.
- Központi szolgáltatások. A magas rendelkezésre állású központi szolgáltatásokat Azure Linux virtuális gépeken a megfelelő magas rendelkezésre állás kiterjesztése a kijelölt terjesztési használt Linux és a magas rendelkezésre állású NFS állomások DRBD tárolási fürt.
- Alkalmazáskiszolgálók. A magas rendelkezésre állás az adatforgalom terheléselosztásával érhető el alkalmazáskiszolgálók egy csoportjában.

### <a name="database-tier"></a>Adatbázis-rétegből

A referencia-architektúrában álló két Azure virtuális gépek magas rendelkezésre állású SAP HANA adatbázis rendszer ábrázol. Az adatbázis-rétegből natív rendszer replikációs szolgáltatás kézi vagy automatikus replikált csomópontok közötti feladatátvételt biztosít:

- Kézi feladatátvételre több HANA-példány telepítése, és HANA rendszer replikációs (HSR) használja.
- Automatikus feladatátvétel a Linux-disztribúció HSR és a Linux magas rendelkezésre állás bővítmény (HAE) használata. Linux HAE szolgáltatásai a fürt HANA erőforrások hibaesemények észlelésére, és a feladatátvétel, a megfelelő csomópont errant szolgáltatások koordinálásához. 

Lásd: [SAP tanúsítványokról és a Microsoft Azure-on futó konfigurációk](/azure/virtual-machines/workloads/sap/sap-certifications).

### <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok
Minden egyes réteg más stratégiával biztosít vészhelyreállítási (DR) védelmet.

- **Alkalmazás-kiszolgálók réteg**. SAP alkalmazáskiszolgálók nem tartalmaz üzleti adatokat. A Azure-ban egy egyszerű vész-Helyreállítási stratégia, ha a SAP alkalmazáskiszolgálók másodlagos régióban, állítsa le. A konfigurációs módosítások vagy az elsődleges kiszolgáló kernel-frissítések esetén a virtuális gépek használata a másodlagos régióba módosítások kell alkalmazni. Például másolja az SAP kernel végrehajtható fájlokat a vész-Helyreállítási virtuális gépek. Automatikus replikációja egy másodlagos régióban alkalmazáskiszolgálók, [Azure Site Recovery](/azure/site-recovery/site-recovery-overview) a javasolt megoldás. A dokumentum írásáig automatikus rendszer-Helyreállítás nem még támogatja a replikálást az elérését gyorsítja fel hálózati konfigurációs beállítás az Azure virtuális gépeken.

- **Központi szolgáltatások**. Ez az összetevő a SAP alkalmazás verem is nem maradnak üzleti adatok. Létrehozhat egy virtuális Gépet a másodlagos régióban a központi szolgáltatások szerepkör futtatásához. Csak a tartalmi szinkronizálni az elsődleges központi szolgáltatások csomópontból-e a /sapmnt megosztást. Is ha a konfigurációs módosítások és a kernel frissítéseket az elsődleges kiszolgálón központi szolgáltatások történik, azok meg kell ismételni a virtuális gép központi szolgáltatásokat futtató másodlagos régióban. A két kiszolgáló szinkronizálásával kapcsolatban vagy Azure Site Recovery használja replikálja a fürtcsomópontokon, vagy egyszerűen csak egy rendszeres, ütemezett feladat segítségével /sapmnt másolja a vész-Helyreállítási oldalon. A build részleteit, másolás és a teszt feladatátvételi folyamat, töltse le a [SAP NetWeaver: a Hyper-V és a Microsoft Azure-alapú vész-helyreállítási megoldást felépítése](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx), és tekintse át a részt 4.3, "SAP SPOF réteg (ASC)." A dokumentum a Windows rendszeren futó NetWeaver vonatkozik, de a Linux-egyenértékű konfigurációt is létrehozhat. Központi szolgáltatások használata [Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview) a a fürtcsomópontok és a tárolási replikálása. Linux hozzon létre egy három csomópontos földrajzi-fürt magas rendelkezésre állás kiterjesztéssel. 

- **Adatbázis-rétegből SAP**. HSR HANA támogatott replikáláshoz használni. Egy helyi, két csomópontos magas rendelkezésre állású összeállításban mellett HSR támogatja a többrétegű replikációs, ahol egy külön Azure-régió, egy harmadik csomópontja egységként idegen, nem része a fürt működik, és a fürtözött HSR párjával másodlagos replikája regisztrálása a replikációs cél. Ez a replikációs lánckapcsolású lánc alkotnak. A feladatátvételt a vész-Helyreállítási csomópontra manuális folyamatban.

Azure Site Recovery segítségével automatikus létrehozása egy teljes mértékben replikált munkakörnyezeti helyet az eredeti, futtatnia kell testre szabott [üzembe helyezési parancsfájlok](/azure/site-recovery/site-recovery-runbook-automation). A Site Recovery először telepíti a virtuális gépek a rendelkezésre állási csoportokban, majd futtat parancsfájlokat, mint az erőforrások hozzáadása a terheléselosztók. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

SAP HANA rendelkezik egy biztonsági mentési szolgáltatás használja az alapul szolgáló Azure infrastruktúra. Annak érdekében, hogy a SAP HANA-adatbázis biztonsági mentése Azure virtuális gépeken futó, a SAP HANA-pillanatkép és az Azure storage pillanatkép használt a biztonsági mentési fájlok konzisztencia biztosításához. További információkért lásd: [biztonsági útmutató SAP Hana Azure virtuális gépeken](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide) és a [Azure Backup szolgáltatás – gyakori kérdések](/azure/backup/backup-azure-backup-faq). Csak HANA egyetlen üzemelő tárolópéldányokat támogatja az Azure storage pillanatkép.

### <a name="identity-management"></a>Identitáskezelés

A központi felügyeleti identitásrendszere minden szinten erőforrásokhoz való hozzáférés szabályozása:

- Azure-erőforrások keresztül hozzáférést biztosítanak [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-control-what-is) (RBAC). 
- Hozzáférés engedélyezése az Azure virtuális gépek LDAP, az Azure Active Directory, a Kerberos vagy a másik rendszerbe. 
- Támogatási hozzáférési jogosultságok magukat az alkalmazások a szolgáltatások SAP biztosít, vagy használjon [OAuth 2.0 és az Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code). 

### <a name="monitoring"></a>Figyelés

Azure biztosít több függvények [figyelése és diagnosztikai](/azure/architecture/best-practices/monitoring) az általános infrastruktúra. Emellett az Azure-beli virtuális gépek (Linux vagy Windows) fejlett monitorozását az Azure Operations Management Suite (OMS) végzi. 

SAP alapú felügyelete, erőforrások és a szolgáltatás teljesítménye az SAP infrastruktúra biztosításához a [Azure SAP fokozott ellenőrzésére](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca) kiterjesztését. Ez a bővítmény betölti az Azure monitorozási statisztikáit az SAP alkalmazásba az operációs rendszer monitorozása és a DBA Cockpit funkcióinak használata céljából. Fokozott SAP figyelési feltétele kötelező Azure SAP futtatásához. További információkért lásd: [SAP Megjegyzés 2191498](https://launchpad.support.sap.com/#/notes/2191498) – "az Azure Linux SAP: figyelés fokozott."

## <a name="security-considerations"></a>Biztonsági szempontok

Az SAP egy saját felhasználófelügyeleti motorral (UME-vel) vezérli a szerepköralapú hozzáféréseket és az engedélyezést az SAP alkalmazáson belül. További információkért lásd: [SAP HANA biztonsági – áttekintés egy](https://archive.sap.com/documents/docs/DOC-62943) (SAP szolgáltatás piactér fiók-hozzáférési szükséges).

A további hálózati biztonság érdekében vegye fontolóra egy [hálózati DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid), a hálózati virtuális készülék használó a tűzfal elé webes kézbesítő és Fiori előfeldolgozó Server címkészletek alhálózatát létrehozásához.

Biztonsági infrastruktúra az adatok titkosítása az átvitel során, és inaktív. A "Biztonsági szempontok" szakaszában a [SAP NetWeaver az Azure virtuális gépek – tervezési és megvalósítási útmutató](/azure/virtual-machines/workloads/sap/planning-guide) történő címet hálózati biztonsági kezdődik, és az S/4HANA vonatkozik. Ez az útmutató adja meg azt is, hogy a tűzfalakon mely portokat kell megnyitni az alkalmazás kommunikációjának lehetővé tételéhez. 

Linux IaaS virtuális gépek lemezeit titkosításához, használhatja a [Azure Disk Encryption](/azure/security/azure-security-disk-encryption). A DM-Crypt funkció a Linux kötettitkosítást biztosít az operációs rendszer és az adatlemezek használ. A megoldás is használható az Azure Key Vault segítségével szabályozhatja, és a lemez-titkosítási kulcsok és titkos kulcsainak kulcstároló-előfizetése kezeléséhez. A virtuális gépek lemezein található, inaktív adatok az Azure Storage-ban titkosítva vannak.

Az SAP HANA inaktív adatainak titkosításához javasoljuk az SAP HANA natív titkosítási technológiájának használatát. 

> [!NOTE]
> Nem a HANA adatokat nyugalmi titkosítást használ Azure Disk Encryption ugyanazon a kiszolgálón. Hana csak a HANA adatok titkosítást használjon.

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

- [A Microsoft Platform blog futó SAP-alkalmazásokból](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Az Azure közösségi támogatás](https://azure.microsoft.com/support/community/)
- [SAP-Közösség](https://www.sap.com/community.html)
- [Stack Overflow](https://stackoverflow.com/tags/sap/)
