---
title: Az SAP NetWeaver és az SAP HANA üzembe helyezése az Azure-ban
description: Bevált eljárások az SAP HANA futtatásához magas rendelkezésre állású környezetben az Azure-ban.
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 33171164c59a520a87ef3209c5bb1b208377221c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/30/2018
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Az SAP NetWeaver és az SAP HANA üzembe helyezése az Azure-ban

Ez a referenciaarchitektúra bemutat néhány bevált eljárást az SAP HANA futtatásához magas rendelkezésre állású környezetben az Azure-ban. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

![0][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

> [!NOTE]
> A referenciaarchitektúra üzembe helyezéséhez SAP-termékek és más, nem a Microsoft által gyártott termékek megfelelő licence szükséges. További információk a Microsoft és az SAP közti partneri kapcsolatról: [SAP HANA az Azure-ban][sap-hana-on-azure].

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

- **Virtuális hálózat (VNet)**. A VNet egy logikailag elkülönített hálózat megjelenítése az Azure-ban. Ezen referenciaarchitektúra összes virtuális gépe ugyanazon a VNeten van üzembe helyezve. A virtuális hálózat további alhálózatokra oszlik. Hozzon létre egy külön alhálózatot mindegyik réteghez, tehát az alkalmazáshoz (SAP NetWeaver), az adatbázishoz (SAP HANA), a felügyelethez (Jumpbox) és az Active Directoryhoz is.

- **Virtuális gépek (VM-ek)**. Ezen architektúra virtuális gépei több, jól elkülöníthető rétegbe vannak csoportosítva.

    - **SAP NetWeaver**. Ide tartozik az SAP ASCS, az SAP Web Dispatcher és az SAP-alkalmazáskiszolgálók. 
    
    - **SAP HANA**. Ebben a referenciaarchitektúrában az adatbázisrétegben az SAP HANA található, amely egyetlen [GS5][vm-sizes-mem] példányon fut. Az SAP HANA tanúsítva van a GS5-ön futó termelési OLAP számítási feladatokhoz vagy a [nagyméretű Azure-példányokon futó SAP HANA-hoz][azure-large-instances]. Ez a referenciaarchitektúra az Azure-beli virtuális gépek G és M sorozataihoz készült. További információk a nagyméretű Azure-példányokon futó SAP HANA-ról: [Az SAP HANA (nagyméretű példányok) áttekintése és architektúrája az Azure-ban][azure-large-instances].
   
    - **Jumpbox**. Más néven bástyagazdagép. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. 
     
    - **Windows Server Active Directory- (AD-) tartományvezérlők**. A tartományvezérlők konfigurálják a Windows Server feladatátvételi fürtöt (lásd az alábbiakban).
 
- **Rendelkezésre állási csoportok**. Helyezze az SAP Web Dispatcher, az SAP-alkalmazáskiszolgáló és az SAP ACSC szerepkörű virtuális gépeket külön rendelkezésre állási csoportokba, és helyezzen üzembe legalább két virtuális gépet minden szerepkörhöz. A virtuális gépek így magasabb szintű szolgáltatói szerződésre (SLA-ra) jogosultak.
    
- **Hálózati adapterek (NIC-k)**. Az SAP NetWeavert és az SAP HANA-t futtató virtuális gépeknek két hálózati adapterrel (NIC-vel) kell rendelkezniük. Mindegyik NIC másik alhálózathoz van rendelve, hogy az adatforgalom különböző típusai el legyenek különítve. További információkért lásd alább a [Javaslatok](#recommendations) részt.

- **Windows Server feladatátvételi fürt**. Az SAP ASCS-t futtató virtuális gépek feladatátvételi fürtként vannak konfigurálva a magas rendelkezésre állás érdekében. A feladatátvételi fürt támogatásához az SIOS DataKeeper Cluster Edition tölti be a fürt megosztott kötete (CSV) szerepét a fürtcsomópontok független lemezeinek replikálásával. További részletekért lásd: [SAP alkalmazások futtatása Microsoft platformon][running-sap].
    
- **Terheléselosztók**. Két [Azure Load Balancert][azure-lb] használunk. Az első, amely az ábra bal oldalán látható, elosztja a forgalmat az SAP Web Dispatcher virtuális gépek között. Ezzel a konfigurációval megvalósul a párhuzamos webes forgalomirányító, amelyet [az SAP Web Dispatcher magas rendelkezésre állásával][sap-dispatcher-ha] kapcsolatos témakör ismertet. A második, jobb oldali terheléselosztó lehetővé teszi a feladatátvételt a Windows Server feladatátvételi fürtben azzal, hogy a bejövő kapcsolatokat az aktív/kifogástalan állapotú csomópontra irányítja.

- **VPN Gateway**. A VPN Gateway kibővíti a helyszíni hálózatot az Azure VNetre. Az ExpressRoute is használható, amely egy dedikált magánhálózati kapcsolatot hoz létre, amely nem a nyilvános interneten keresztül valósul meg. A példában szereplő megoldás nem telepíti az átjárót. További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz][hybrid-networking].

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="load-balancers"></a>Terheléselosztók

Az [SAP Web Dispatcher][sap-dispatcher] végzi a HTTP(S)-forgalom terheléselosztását a kettős veremmel rendelkező kiszolgálók (ABAP és Java) felé. Az SAP évek óta az egyvermes alkalmazáskiszolgálók használatát népszerűsíti, ezért manapság nagyon kevés alkalmazás fut kettős vermű üzembehelyezési modelleken. Az ábrán látható Azure Load Balancer implementálja az SAP Web Dispatcher magas rendelkezésre állású fürtjét.

Az alkalmazáskiszolgálók felé irányuló forgalom terheléselosztása az SAP-n belül történik. Az SAP-kiszolgálóhoz DIAG és RFC-k útján csatlakozó SAPGUI-ügyfelektől érkező forgalom terheléselosztásához az SCS-üzenetkiszolgáló SAP App Server [bejelentkezési csoportokat][logon-groups] hoz létre. 

Az SMLG egy SAP ABAP-tranzakció, amelynek célja az SAP Central Services bejelentkezési terheléselosztási képességének felügyelete. A bejelentkezési csoport háttérkészletéhez több ABAP-alkalmazáskiszolgáló tartozik. Az ASCS fürtszolgáltatásokhoz hozzáférő ügyfelek az Azure Load Balancerhez egy előtérbeli IP-címen keresztül csatlakoznak. Az ASCS fürt virtuális hálózati nevéhez szintén tartozik egy IP-cím. Lehetőség van ezt a címet társítani egy további IP-címhez az Azure Load Balancerön, hogy a fürt távolról is felügyelhető legyen.  

### <a name="nics"></a>Hálózati adapterek (NIC-k)

Az SAP környezetkezelési funkcióihoz a kiszolgáló adatforgalmát szét kell választani különböző hálózati adapterekre. Például az üzleti adatokat külön kell választani az adminisztrációs forgalomtól és a biztonsági mentések forgalmától. Azzal, hogy a különböző alhálózatokhoz több hálózati adaptert rendelünk, lehetővé tesszük az adatok ily módon való szétválasztását. További információkért lásd a „Hálózat” témakört a [Magas rendelkezésre állás létrehozása az SAP NetWeaver és az SAP HANA számára][sap-ha] című PDF-dokumentumban.

Rendelje hozzá az adminisztrációs NIC-t a felügyeleti alhálózathoz, az adatkommunikációs NIC-t pedig egy külön alhálózathoz. Konfigurációs részletek: [Több hálózati adapterrel rendelkező Windows rendszerű virtuális gép létrehozása és felügyelete][multiple-vm-nics].

### <a name="azure-storage"></a>Azure Storage

Javasoljuk minden adatbázis-kiszolgáló virtuális géphez az Azure Premium Storage használatát az egyenletes olvasási/írási késleltetések érdekében. Az SAP-alkalmazáskiszolgálókhoz, például az (A)SCS virtuális gépekhez az Azure Standard Storage is használható, mivel az alkalmazás végrehajtása a memóriában történik, a lemezt pedig csak naplózásra használja a rendszer.

A legjobb megbízhatóság érdekében javasoljuk az [Azure Managed Disks][managed-disks] használatát. A felügyelt lemezek biztosítják, hogy a rendelkezésre állási csoportban elkülönüljenek egymástól a virtuális gépek lemezei a kritikus hibapontok elkerülése érdekében.

> [!NOTE]
> A referenciaarchitektúra Resource Manager-sablonja jelenleg nem használ felügyelt lemezeket. Tervezzük a sablont frissítését a felügyelt lemezek használatára.

A magas IOPS és a lemezsávszélesség magas átviteli sebessége érdekében az Azure Storage elrendezésére is érvényesek a tárterület-teljesítmény optimalizálására szolgáló ajánlott gyakorlatok. Például több lemez összevonása és ezzel egy nagyobb lemezkötet létrehozása növeli az I/O-teljesítményt. A tárolóban található ritkán változó tartalmak olvasási gyorsítótárazásának engedélyezésével növelhető az adatelérés sebessége. További részletekért a teljesítménykövetelményekkel kapcsolatban lásd az [1943937-es számú SAP-jegyzetet a hardverkonfiguráció-ellenőrzési eszközről][sap-1943937].

Az adattár biztonsági mentéséhez javasoljuk a [ritkán használt adatok tárolási rétegét][cool-blob-storage] az Azure Blob Storage-ban. A ritkán használt adatok tárolási rétegében költséghatékonyan tárolhatók a ritkábban használt, hosszú élettartamú adatok.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az SAP alkalmazásrétegben az Azure számos különböző méretet kínál a virtuális gépek felskálázásához. A teljes listát lásd az [1928533-as számú SAP-jegyzetben – SAP alkalmazások az Azure-ban: Támogatott termékek és Azure-virtuálisgéptípusok][sap-1928533]. A horizontális felskálázáshoz adjon hozzá további virtuális gépeket a rendelkezésre állási csoporthoz.

Az Azure-alapú virtuális gépeken futó SAP HANA (és mind az OLTP, mind az OLAP SAP-alkalmazások) esetében az SAP által tanúsított virtuálisgép-méret a GS5 egyetlen virtuálisgép-példánnyal. A nagyobb méretű számítási feladatokhoz a Microsoft [nagyméretű Azure-példányokat][azure-large-instances] is kínál a Microsoft Azure Certified programban részt vevő adatközpontokban található fizikai kiszolgálókon futó SAP HANA-hoz. A nagyméretű példányok jelenleg példányonként akár 4 TB memóriakapacitást is nyújtanak. Többcsomópontos konfigurációval a teljes memóriakapacitás akár 32 TB-ra növelhető.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az SAP-alkalmazás egy központi adatbázisban futó elosztott telepítése esetében az alapul szolgáló telepítés replikálásával érhető el a magas rendelkezésre állás. A magas rendelkezésre állás kialakítása az architektúra minden rétegében eltérő:

- **Web Dispatcher.** A magas rendelkezésre állás az SAP-alkalmazások forgalmához használt redundáns SAP Web Dispatcher-példányokkal érhető el. Lásd az [SAP Web Dispatcher][swd] című részt az SAP dokumentációjában.

- **ASCS.** Az Azure-beli Windows rendszerű virtuális gépeken futó ASCS magas rendelkezésre állásának eléréséhez a Windows Server feladatátvételi fürtszolgáltatást használhatjuk a fürt megosztott kötetét adó SIOS DataKeeperrel együtt. A megvalósítás részleteit lásd: [Az Azure-ban futó SAP ASCS fürtözése][clustering].

- **Alkalmazáskiszolgálók.** A magas rendelkezésre állás az adatforgalom terheléselosztásával érhető el alkalmazáskiszolgálók egy csoportjában.

- **Adatbázisréteg.** Ez a referenciaarchitektúra egyetlen SAP HANA-adatbázispéldányt helyez üzembe. Magas rendelkezésre álláshoz helyezzen üzembe több példányt, majd használja a HANA System Replication (HSR) eszközt a manuális feladatátvétel implementálásához. Az automatikus feladatátvétel engedélyezéséhez egy HA bővítmény szükséges az adott Linux-disztribúcióhoz.

### <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

Minden egyes réteg más stratégiával biztosít vészhelyreállítási (DR) védelmet.

- **Alkalmazáskiszolgálók.** Az SAP-alkalmazáskiszolgálók nem tartalmaznak üzleti adatokat. Az Azure-ban egyszerű vészhelyreállítási stratégia lehet, ha az SAP-alkalmazáskiszolgálókat egy másik régióban hozzuk létre. Ha az elsődleges alkalmazáskiszolgálón bármilyen konfigurációmódosítás vagy kernelfrissítés lép életbe, a módosításokat át kell másolni a vészhelyreállítás régióban található virtuális gépekre. Ilyenek például a kernel végrehajtható fájljai.

- **SAP Central Services.** Az SAP alkalmazáscsoport ezen összetevője szintén nem tárol üzleti adatokat. Létrehozhat a DR régióban egy virtuális gépet, amely az SCS szerepkört futtatja. Az elsődleges SCS csomópont egyedüli szinkronizálandó tartalma az **/sapmnt** megosztási tartalom. Emellett ha az elsődleges SCS-kiszolgálókon bármilyen konfigurációmódosítás vagy kernelfrissítés történik, azokat a DR SCS-kiszolgálón is meg kell ismételni. A két kiszolgáló szinkronizálását egy egyszerű ütemezett másolási feladattal elvégezheti, amely átmásolja az **/sapmnt** tartalmát a DR oldalra. A létrehozási, másolási és tesztelési feladatátvételi folyamatok részleteiért töltse le az [SAP NetWeaver: Building a Hyper-V and Microsoft Azure–based Disaster Recovery Solution][sap-netweaver-dr] (SAP NetWeaver: Hyper-V- és Microsoft Azure-alapú vészhelyreállítási megoldás létrehozása) című dokumentumot, és olvassa el a „4.3: SAP SPOF layer (ASCS)” (SAP SPOF réteg – ASCS) fejezetet.

- **Adatbázisréteg.** Használjon a HANA által támogatott replikációs megoldást, például a HSR-t vagy a tárreplikációt. 

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az SAP HANA rendelkezik egy biztonsági mentési funkcióval, amely az alapul szolgáló Azure-infrastruktúrát használja. Azure virtuális gépen futó SAP HANA-adatbázis biztonsági mentéséhez az SAP HANA pillanatképe és az Azure-tároló pillanatképe is szükséges a biztonsági mentési fájlok konzisztenciájának biztosítása érdekében. Részletekért lásd az [SAP HANA az Azure-beli virtuális gépeken – Biztonsági mentési útmutató][hana-backup] és az [Azure Backup szolgáltatás – gyakori kérdések][backup-faq] cikkeket.

Az Azure számos különféle funkciót nyújt az infrastruktúra átfogó [monitorozásához és diagnosztizálásához][monitoring]. Emellett az Azure-beli virtuális gépek (Linux vagy Windows) fejlett monitorozását az Azure Operations Management Suite (OMS) végzi.

Ha SAP-alapú monitorozásra van szüksége az erőforrásokhoz és az SAP-infrastruktúra szolgáltatási teljesítményéhez, használja az Azure SAP Enhanced Monitoring bővítményt. Ez a bővítmény betölti az Azure monitorozási statisztikáit az SAP alkalmazásba az operációs rendszer monitorozása és a DBA Cockpit funkcióinak használata céljából. 

## <a name="security-considerations"></a>Biztonsági szempontok

Az SAP egy saját felhasználófelügyeleti motorral (UME-vel) vezérli a szerepköralapú hozzáféréseket és az engedélyezést az SAP alkalmazáson belül. Részletekért lásd: [SAP HANA – Biztonsági áttekintés][sap-security]. (A hozzáféréshez egy SAP Service Marketplace-fiók szükséges.)

Az infrastruktúra biztonsága érdekében az adatok a továbbítás és a tárolás közben is védelem alatt állnak. A hálózati biztonság témájával az [SAP NetWeaver az Azure-beli virtuális gépeken – Tervezési és megvalósítási útmutató][netweaver-on-azure] „Biztonsági szempontok” szakasza foglalkozik. Ez az útmutató adja meg azt is, hogy a tűzfalakon mely portokat kell megnyitni az alkalmazás kommunikációjának lehetővé tételéhez. 

Windows és Linux rendszerű IaaS virtuális gépek lemezeinek titkosításához használja az [Azure Disk Encryption][disk-encryption] szolgáltatást. Az Azure Disk Encryption Windows rendszeren a BitLocker funkcióval, Linux rendszeren pedig a DM-Crypt funkcióval titkosítja az operációs rendszer és az adatlemezek köteteit. A megoldás az Azure Key Vaulttal együtt is használható, így a lemeztitkosítási kulcsokat és titkos kulcsokat a Key Vault-előfizetésből vezérelheti és felügyelheti. A virtuális gépek lemezein található, inaktív adatok az Azure Storage-ban titkosítva vannak.

Az SAP HANA inaktív adatainak titkosításához javasoljuk az SAP HANA natív titkosítási technológiájának használatát.

> [!NOTE]
> Egy adott kiszolgálón ne használja egyszerre a HANA inaktívadat-titkosítási funkcióját és az Azure Disk Encryptiont.

A VNet különböző alhálózatai között érdemes lehet [hálózati biztonsági csoportok][nsg] (NSG-k) használatával korlátozni az adatforgalmat.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése 

A referenciaarchitektúra üzembehelyezési szkriptjei elérhetők a [GitHubon][github].


### <a name="prerequisites"></a>Előfeltételek

- A telepítés végrehajtásához hozzáféréssel kell rendelkeznie a SAP Software Download Center letöltőközponthoz.
 
- Telepítse az [Azure PowerShell][azure-ps] legújabb verzióját. 

- Ehhez az üzemelő példányhoz 51 mag szükséges. Az üzembe helyezése előtt ellenőrizze, hogy az előfizetése virtuálisgép-magokra vonatkozó kvótája elegendő-e. Ha nem, küldjön támogatási kérelmet az Azure Portalról további kvótáért.
 
- Ez az üzemelő példány egy GS sorozatú virtuális gépet használ. A GS sorozat régiónkénti elérhetőségét [itt ellenőrizheti][region-availability].

- Az üzembe helyezés költségeinek meghatározásához lásd: [Azure-díjkalkulátor][azure-pricing]. 
 
A referenciaarchitektúra a következő virtuális gépeket helyezi üzembe:

| Erőforrás neve | Virtuális gép mérete | Cél  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP Central Services |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | SAP NetWeaver alkalmazás |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | SAP HANA-adatbázispéldány |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

Egyetlen SAP HANA-példány van üzembe helyezve. Az alkalmazások virtuális gépeinek esetében az üzembe helyezendő példányokat a sablonparaméterek határozzák meg.

### <a name="deploy-sap-infrastructure"></a>SAP-infrastruktúra üzembe helyezése

Az architektúra növekményesen vagy egyszerre is telepíthető. Első alkalommal növekményes üzembe helyezést javaslunk, hogy megismerhesse, az üzembe helyezés melyik lépés milyen művelet hajtható végre. A növekmény az alábbi *mód* paraméterek egyikével határozható meg

| Mód           | Művelet                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastruktúra | A hálózati infrastruktúra üzembe helyezése az Azure-ban.        |
| számítási feladat       | Az SAP-kiszolgálók üzembe helyezése a hálózaton.             |
| összes            | Üzembe helyezi az összes fenti környezetet.              |

A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket:

1. Töltse le vagy klónozza a [GitHub-adattárat][github] a saját helyi számítógépére.

2. Nyisson meg egy PowerShell-ablakot, és lépjen a `/sap/sap-hana/` mappára.

3. Futtassa az alábbi PowerShell-parancsmagot. A `<subscription id>` paraméterhez adja meg az Azure-előfizetése azonosítóját. A `<location>` paraméter esetében adjon meg egy Azure-régiót (pl. `eastus` vagy `westus`). A `<mode>` paraméternél adja meg a fent említett módok egyikét.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  Ha a rendszer kéri, jelentkezzen be az Azure-fiókjába. 

A kiválasztott módtól függően az üzembehelyezési szkriptek befejezése több órát is igénybe vehet.

> [!WARNING]
> A paraméterfájlok több helyen nem módosítható jelszót (`AweS0me@PW`) tartalmaznak. Módosítsa ezeket az értékeket az üzembe helyezés előtt.
 
### <a name="configure-sap-applications-and-database"></a>SAP-alkalmazások és -adatbázis konfigurálása

Az SAP-infrastruktúra üzembe helyezését követően telepítse és konfigurálja az SAP-alkalmazásait és HANA-adatbázisát a virtuális gépeken az alább ismertetett módon.

> [!NOTE]
> Az SAP telepítésével kapcsolatos utasításokhoz rendelkeznie kell felhasználónévvel és jelszóval az SAP támogatási portáljához, hogy letölthesse az [SAP telepítési útmutatóit][sap-guide].

1. Jelentkezzen be a jumpboxba (`ra-sap-jumpbox-vm1`). A jumpbox használatával jelentkezhet majd be a többi virtuális gépbe. 

2.  Minden `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` nevű virtuális gép esetében jelentkezzen be a virtuális gépbe, majd telepítse és konfigurálja az SAP Web Dispatcher-példányt [a Web Dispatcher telepítését][sap-dispatcher-install] ismertető wikiben leírt lépések segítségével.

3.  Jelentkezzen be az `ra-sap-data-vm1` nevű virtuális gépbe. Telepítse és konfigurálja az SAP Hana-adatbázispéldányt az [SAP HANA-kiszolgáló telepítési és frissítési útmutatójának][hana-guide] segítségével.

4. Minden `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` nevű virtuális gép esetében jelentkezzen be a virtuális gépbe, majd telepítse és konfigurálja az SAP Central Services (SCS) szolgáltatást az [SAP telepítési útmutatóinak][sap-guide] használatával.

5.  Minden `ra-sapApps-vm1` ... `ra-sapApps-vmN` nevű virtuális gép esetében jelentkezzen be a virtuális gépbe, majd telepítse és konfigurálja az SAP NetWeaver alkalmazást az [SAP telepítési útmutatóinak][sap-guide] használatával.

**_A referenciaarchitektúrában közreműködtek_** &mdash;: Rick Rainey, Ross Sponholtz, Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.blob.core.windows.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "SAP HANA-architektúra a Microsoft Azure használatával"
