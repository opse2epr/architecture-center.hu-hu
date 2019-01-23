---
title: Nagyméretű Azure-példányokon futó SAP HANA futtatása
titleSuffix: Azure Reference Architectures
description: Bevált eljárások az SAP HANA futtatásához magas rendelkezésre állású környezetben az nagyméretű Azure-példányokon.
author: lbrader
ms.date: 05/16/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, SAP
ms.openlocfilehash: 711e233de534597f9cd06ccaa95481a51acc4468
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485341"
---
# <a name="run-sap-hana-on-azure-large-instances"></a>Nagyméretű Azure-példányokon futó SAP HANA futtatása

Ez a referenciaarchitektúra az Azure-ban (nagyméretű példányok) SAP HANA futtatásához magas rendelkezésre állás és vészhelyreállítás (DR) bevált eljárásokat mutat be. HANA nagyméretű példányok nevű, ezt az ajánlatot a fizikai kiszolgálók Azure-régióban üzemel.

![Nagyméretű példányok az Azure használatával SAP HANA-architektúra](./images/sap-hana-large-instances.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

> [!NOTE]
> A referenciaarchitektúra üzembe helyezéséhez SAP-termékek és más, nem a Microsoft által gyártott termékek megfelelő licence szükséges.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő infrastruktúra-összetevőkből áll.

- **Virtuális hálózat**. A [Azure Virtual Network] [ vnet] szolgáltatás biztonságosan köti össze az Azure-erőforrások egymáshoz, és van felosztva, külön [alhálózatok] [ subnet] az egyes rétegekhez. SAP alkalmazási rétegekben vannak üzembe helyezve az előzménylistákat az Azure gépeken (VM) a HANA-adatbázis elhelyezkedhet a nagyméretű példányok réteg csatlakozni.

- **Virtuális gépek**. A SAP alkalmazás réteg és a megosztott szolgáltatások réteg virtuális gépek használhatók. Ez utóbbi tartalmazza a jumpbox nagyméretű HANA-példányok beállítása és más virtuális gépek hozzáférést biztosítanak a rendszergazdák által használt.

- **Nagyméretű HANA-példány**. A [fizikai kiszolgáló] [ physical] certified a t futtatja SAP HANA SAP HANA szabott adatközpont integrációja (TDI) szabványoknak való megfelelést. Ez az architektúra két HANA nagyméretű példányok használ: egy elsődleges és másodlagos számítási egység. Az adatok rétege magas rendelkezésre állást biztosítunk HANA rendszer replikációs (HSR) keresztül.

- **Magas rendelkezésre állású párként**. Nagyméretű HANA-példányokhoz panelek egy csoportját együtt felügyelt alkalmazás redundancia és megbízhatóság biztosításához.

- **(A Microsoft vállalati peremhálózati) MSEE**. MSEE egy szolgáltatáskapcsolati pont kapcsolati vagy ExpressRoute-kapcsolatcsoport segítségével a peremhálózatban.

- **Hálózati adapterek (NIC)**. Ahhoz, hogy a kommunikáció, a nagyméretű HANA-példányt kiszolgáló alapértelmezés szerint négy virtuális hálózati adapter biztosít. Ez az architektúra egy hálózati adapter közötti kommunikáció, egy második hálózati adapter szükséges HSR, egy külső hálózati adapter nagyméretű HANA-példány tárolás és a egy negyedik a fürtszolgáltatás magas rendelkezésre állású használja az iSCSI-csomópontok csatlakoztatásához szükséges.

- **A hálózati fájlrendszerben (NFS) storage**. A [NFS] [ nfs] server támogatja a védett adatok megőrzése nagyméretű HANA-példány biztosító hálózati fájlmegosztáson.

- **Az ExpressRoute**. [Az ExpressRoute] [ expressroute] a javasolt az Azure hálózati szolgáltatás a helyszíni hálózat és az Azure virtuális hálózatok, amelyek nem a nyilvános interneten haladnak át közötti magánhálózati kapcsolatokat hozhat létre. Az Azure virtuális gépek HANA nagyméretű példányok egy másik az ExpressRoute-kapcsolat használatával csatlakozni. Az ExpressRoute-kapcsolat az Azure virtuális hálózat és a nagyméretű HANA-példányok között a Microsoft ajánlat részeként van beállítva.

- **Átjáró**. Az ExpressRoute-átjárót az Azure virtuális hálózat, az SAP alkalmazásrétegre a nagyméretű HANA-példány hálózati használt kapcsolódásra szolgál. Használja a [nagy teljesítményű és Ultranagy teljesítményű] [ sku] Termékváltozat.

- **A vészhelyreállítás (DR)**. Kérésre a tárreplikáció támogatott, és akkor konfigurálhatók, hogy az elsődleges kiszolgálóról a [DR hely] [ DR-site] egy másik régióban található.

## <a name="recommendations"></a>Javaslatok

Követelmények is eltérőek lehetnek, ezért ezeket a javaslatokat kiindulópontként.

### <a name="hana-large-instances-compute"></a>Nagyméretű HANA-példányok számítási

[Nagyméretű példányok] [ physical] fizikai kiszolgálókat az Intel EX E7 CPU architektúra alapján, és a egy nagyméretű szolgáltatáspéldányban konfigurálva –, egy meghatározott készletének kiszolgálók vagy a többi panelen. Egy számítási egység egyenlő egy kiszolgáló vagy panelen, és a egy stamp tevődik össze több kiszolgálóhoz vagy a többi panelen. Egy nagyméretű szolgáltatáspéldányban kiszolgálók nem megosztott, és egy ügyfél központi telepítését az SAP HANA futtatása dedikált.

SKU-k különböző HANA nagyméretű példányokhoz, 20 TB-ig egyetlen példányra (60 TB-os horizontális felskálázás) memóriát az S/4HANA vagy egyéb SAP HANA számítási feladatainak támogatása érhetők el. [Két osztályba] [ classes] kiszolgálók érhetők el:

- I. osztály típusa: S72, S72m, S144, S144m, S192 és S192m

- Írja be a II osztály: S384, S384m, S384xm, S576m, S768m, and S960m

Például a S72 Termékváltozat 768 GB RAM, a tárolás és 2 Intel Xeon processzorok (E7-8890 v3) 36 maggal 3 terabájt (TB) tartalmaz. Az architektúra és kialakítás munkamenetek során meghatározott méretezési követelményeinek SKU kiválasztása. Mindig győződjön meg arról, hogy a méretezési vonatkozik-e a megfelelő Termékváltozatot. Képességek és központi telepítésére vonatkozó követelmények [típusa szerint változó][type], és a rendelkezésre állási eltérő [régió][region]. Akkor is is fokozzák a Termékváltozat a nagyobb termékváltozatra.

A Microsoft segíti a nagyméretű példány beállítása, de a felelőssége, hogy az operációs rendszer konfigurációs beállításainak ellenőrzése. Ellenőrizze, hogy tekintse át a pontos Linux kiadásban a legújabb SAP-megjegyzések.

### <a name="storage"></a>Storage

A tárolási elrendezés az SAP Hana TDI a ajánlása szerint van megvalósítva. Nagyméretű HANA-példányok egy adott tárolási konfigurációt, a standard szintű TDI leírásában kapható. Azonban további tárterületet 1 TB-os lépésekben vásárolhat.

Üzleti szempontból kritikus környezetek, például a gyors helyreállítás követelményeinek kielégítése érdekében NFS használt, és nem közvetlenül csatlakoztatott tárolók. Az NFS-tároló kiszolgáló nagyméretű HANA-példányokhoz több-bérlős környezetben, ahol a bérlők elkülönített és védi a számítási, hálózati és tárolási elkülönítési üzemel.

Támogatja a magas rendelkezésre állás az elsődleges helyen, használjon másik tárolási elrendezést. A több gazdagép kibővített, például a storage van osztva. Egy másik magas rendelkezésre állású lehetőség például a HSR-beli alkalmazásalapú replikáció. A Vészhelyreállítás azonban egy pillanatkép-alapú tárreplikáció használnak.

### <a name="networking"></a>Hálózat

Ez az architektúra és a fizikai, mind a virtuális hálózatokat használ. A virtuális hálózat része az Azure IaaS és a egy különálló HANA nagyméretű példányok fizikai hálózaton keresztül kapcsolódik [ExpressRoute][expressroute]] Kapcsolatcsoportok. Létesítmények közötti átjáró csatlakozik a helyszíni hely a számítási feladatokat az Azure virtuális hálózatban.

Nagyméretű HANA-példányokhoz hálózatok különítve egymástól a biztonság. Különböző régiókban levő példányok nem kommunikálnak egymással, a dedikált tárreplikáció kivételével. Azonban a HSR használatához régiók közötti kommunikáció szükség. [IP-útválasztási táblázatok] [ ip] vagy proxyk engedélyezése több régiót HSR használható.

Minden Azure virtuális hálózatok, amelyek egy adott régióban nagyméretű HANA-példányokhoz csatlakoznak lehetnek [közötti csatlakozó] [ cross-connected] HANA nagyméretű példányok egy másodlagos régióban az expressroute-on keresztül.

HANA nagyméretű példányok az ExpressRoute üzembe helyezés során alapértelmezés szerint tartalmazza. A telepítő egy adott hálózati elrendezés szükséges, beleértve a szükséges cím CIDR-tartományok és az útválasztási tartomány. További információkért lásd: [SAP HANA (nagyméretű példányok) infrastruktúra és kapcsolódás az Azure-ban][HLI-infrastructure].

## <a name="scalability-considerations"></a>Méretezési szempontok

A kisebbre vagy nagyobbra méretezhetők, a nagyméretű HANA-példányok rendelkezésre álló kiszolgálók számos méretek közül választhat. Kategóriába sorolt [i. típus és II. típusú] [ classes] és a különböző számítási feladatokhoz igazított. Válasszon, amely a következő három év növelhető az alkalmazások és szolgáltatások-méretet. Egy éves kötelezettségvállalás is elérhetők.

Egy több gazdagépet bővítő telepítés BW/4HANA-telepítések általában használják, egyfajta adatbázis particionálási stratégia. A horizontális felskálázáshoz tervezze meg a telepítés előtt HANA táblák elhelyezését. -Infrastruktúra szempontból több gazdagép csatlakozik egy megosztott tároló kötet, abban az esetben, ha meghiúsul egy számítási munkavégző csomópontja között a HANA system, amely lehetővé teszi, hogy gyors átvétel készenléti gazdagépek.

S/4hana-t és az SAP Business Suite on HANA az egyetlen panelen lehet méretezve akár 20 TB egyetlen nagyméretű HANA-példányokhoz példánnyal.

Előzmények nélküli forgatókönyvek esetén a [SAP gyors Szimbólumméretező] [ quick-sizer] HANA épülő SAP-szoftverek végrehajtásának memóriaigényének kiszámításához érhető el. HANA memóriakövetelményei növelje az adatmennyiség növekedésével. A rendszer aktuális memóriafogyasztás alapjaként használni a jövőbeli fogyasztás előrejelzésére szolgáló, és a egy a nagyméretű HANA-példányok méretei képezze le az igény szerinti.

Ha már rendelkezik az SAP üzemelő példányok, SAP biztosít a jelentések segítségével ellenőrizze a meglévő rendszerek által használt adatokat, majd számítja ki a memória a HANA-példány. Például tekintse meg az alábbi SAP-megjegyzések:

- SAP-Jegyzetnek [1793345] [ sap-1793345] – SAP Suite on HANA méretezése
- SAP-Jegyzetnek [1872170] [ sap-1872170] -Suite on HANA, és S/4 HANA méretezési jelentés
- SAP-Jegyzetnek [2121330] [ sap-2121330] – gyakori kérdések: SAP BW on HANA jelentés méretezése
- SAP-Jegyzetnek [1736976] [ sap-1736976] -jelentés BW on HANA méretezése
- SAP-Jegyzetnek [2296290] [ sap-2296290] – új jelentés BW on HANA méretezése

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás-redundanciát a magas rendelkezésre állású infrastruktúra-megoldások az általános témát. Az olyan vállalatok, amelyek kevésbé szigorú garantált szolgáltatási szint egypéldányos Azure virtuális gépek kínálnak hasznos üzemidővel SLA-t. További információkért lásd: [Azure szolgáltatásiszint-szerződés](https://azure.microsoft.com/support/legal/sla/).

Működik együtt az SAP, a rendszerintegrátor, vagy a Microsoft megfelelően tervezhet és végrehajtásához egy [magas rendelkezésre állású és vész-helyreállítási] [ hli-hadr] stratégia. Ez az architektúra a következő Azure [szolgáltatásiszint-szerződés] [ sla] (SLA) Hana az Azure-ban (nagyméretű példányok). A rendelkezésre állási követelmények elemzése, fontolja meg a hiba, a hasznos üzemidő szolgáltatásokhoz kívánt szintjét és a gyakori metrikák minden olyan hibaérzékeny pontokat:

- A helyreállítási időre vonatkozó célkitűzés (RTO) azt jelenti, az időtartam, ahol a HANA nagyméretű példányok kiszolgáló nem érhető el.

- A helyreállítási időkorlátot (RPO) azt jelenti, hogy mely ügyfelek adatvesztés történhet egy hiba miatt a maximális tolerálható időszak.

Magas rendelkezésre állás érdekében egy magas rendelkezésre ÁLLÁS pár egynél több példány üzembe helyezése, és a HSR használata egy szinkron módban adatveszteség és állásidő minimalizálása érdekében. Mellett egy helyi, kétcsomópontos magas rendelkezésre állású összeállításban HSR támogatja a többrétegű replikációs, ahol egy külön Azure-régióban egy harmadik csomóponton regisztrálja a fürtözött HSR pár replikációs célhelye, a másodlagos replikára. Ez képezi egy replikációs lánckapcsolt. A feladatátvételt, hogy a DR-csomópont manuális folyamat során a rendszer.

HANA nagyméretű példányok HSR automatikus feladatátvétellel beállításakor beállítása a Microsoft-kezelési csapatunk kérheti egy [STONITH eszköz] [ stonith] a meglévő kiszolgálói számára.

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

Ez az architektúra támogatja [vész-helyreállítási] [ hli-dr] HANA nagyméretű példányok különböző Azure-régiók között. A nagyméretű HANA-példányokhoz DR támogatásához két módja van:

- Tárolóreplikáció. Elsődleges tároló tartalma folyamatosan replikálódnak a távoli DR adattároló rendszerek, amelyek a kijelölt kiszolgálón DR HANA nagyméretű példányok elérhető. A storage-bA a HANA-adatbázis nem betölti a memóriába. A DR-beállítás esetén egyszerűbb felügyeleti szempontból. Annak megállapításához, hogy ez-e a megfelelő stratégiát, érdemes lehet az adatbázis betöltési idő, szemben a rendelkezésre állási SLA-t. Tárreplikáció emellett lehetővé teszi időpontban a helyreállítás végrehajtásához. Ha többcélú (költség optimalizált) DR be van állítva, meg kell vásárolnia a DR helyen azonos méretű tárhely. A Microsoft biztosít bontaniuk [tárolási pillanatkép és a feladatátvételi parancsfájl] [ scripts] HANA feladatátvételi a HANA nagyméretű példányok ajánlat részeként.

- Többrétegű HSR (ahol a HANA-adatbázis alakzatot memória van betöltve) DR régióban egy harmadik replikával. Ezt a beállítást támogatja a helyreállítás gyorsabb, de nem támogatja a időponthoz helyreállítás. HSR egy másodlagos rendszert igényel. A DR hely HANA-rendszerreplikálást proxyk, például az nginx-et vagy az IP-táblák kezeli.

> [!NOTE]
> Ez a referenciaarchitektúra az költségek optimalizálhatja egypéldányos környezetben futtatja. Ez [költség optimalizált forgatókönyv](https://blogs.sap.com/2016/07/19/new-whitepaper-for-high-availability-for-sap-hana-cost-optimized-scenario/) a nem éles HANA feladat ellátására alkalmas.

## <a name="backup-considerations"></a>Biztonsági másolat szempontjai

Több rendelkezésre álló lehetőségek közül választhat az üzleti követelmények alapján, [biztonsági mentési és helyreállítási][hli-backup].

| Ez a beállítás                   | Szakemberek számára                                                                                                   | Hátrányok                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| HANA biztonsági mentés        | Natív SAP. Beépített konzisztencia-ellenőrzést.                                                             | Mennyi ideig biztonsági mentési és helyreállítási idő. Tárhelyhasználat terület. |
| HANA pillanatképe      | Natív SAP. Gyors mentését és helyreállítását.                                                               |                                       |
| Storage-pillanatkép   | Nagyméretű HANA-példányokhoz is. Optimalizált DR HANA nagyméretű példányokhoz. Rendszerindító kötet biztonsági mentését támogató. | Kötetenként legfeljebb 254 pillanatképeket.                          |
| Naplóalapú biztonsági mentés         | Szükséges idő a helyreállítási pontot.                                                                   |                                                            |
| Egyéb biztonsági mentési eszközök | Georedundáns biztonsági másolat helyét.                                                                             | További licencelési költségeit.                                |

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Nagyméretű HANA-példányokhoz erőforrások, például a CPU, memória, hálózati sávszélességet és tárolóhelyet, SAP, HANA Studio, az SAP HANA vezérlőpultja, SAP-megoldás Manager és más natív Linux-eszközök használatával figyelheti. Nagyméretű HANA-példányokhoz nem biztosítja a beépített monitorozási eszközökkel. A Microsoft kínál forrásanyagok [hibaelhárításához és monitorozásához] [ hli-troubleshoot] a szervezet elvárásainak, és a Microsoft támogatási csapatunk segítségére a technikai problémák elhárítása.

Ha további számítási képesség, be kell szereznie egy nagyobb Termékváltozat.

## <a name="security-considerations"></a>Biztonsági szempontok

- Alapértelmezés szerint a HANA nagyméretű példányok használják a tárolás titkosítása a TDE (transzparens adattitkosítás) inaktív adatok alapján.

- Nagyméretű HANA-példányok és a virtuális gépek között átvitt adatok nincsenek titkosítva. Az adatátvitel titkosítását, az alkalmazás-specifikus titkosítást. Tekintse meg az SAP-Jegyzetnek [2159014] [ sap-2159014] – gyakori kérdések: Az SAP HANA biztonsági.

- Elkülönítés a nagyméretű HANA-példány több-bérlős környezetben a bérlők közötti biztonságot nyújt. Bérlők elkülönülnek a saját VLAN használatával.

- [Azure-beli hálózati biztonsági eljárások] [ network-best-practices] hasznos útmutatást nyújtanak.

- Csakúgy, mint bármely üzemelő [operációs rendszer korlátozásának] [ os-hardening] ajánlott.

- A fizikai biztonság érdekében az Azure-adatközpontokhoz, korlátozott csak az arra jogosult személyek. Nincsenek felhasználók férhetnek hozzá a fizikai kiszolgálók.

További információkért lásd: [SAP HANA biztonsági – áttekintés][sap-security]. () Egy SAP Service Marketplace-fiók megadása kötelező a hozzáféréshez)

## <a name="communities"></a>Közösségek

A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

- [A Microsoft Platform Blog futó SAP-alkalmazások][running-sap-blog]
- [Azure közösségi támogatás][azure-forum]
- [Az SAP közösségi][sap-community]
- [A stack Overflow SAP][stack-overflow]

## <a name="related-resources"></a>Kapcsolódó erőforrások

Tekintse át az alábbiakat érdemes [Azure példaforgatókönyvek](/azure/architecture/example-scenario) , amelyek bemutatják, hogy egyes technológiákat használó adott megoldások:

- [Az SAP számítási feladatok futtatása Azure-beli Oracle-adatbázis használata](/azure/architecture/example-scenario/apps/sap-production)
- [Az SAP-feladatokat az Azure-ban fejlesztési és tesztelési környezetek](/azure/architecture/example-scenario/apps/sap-dev-test)

<!-- links -->

[azure-forum]: https://azure.microsoft.com/support/forums/
[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[classes]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[cross-connected]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[dr-site]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[expressroute]: /azure/architecture/reference-architectures/hybrid-networking/expressroute
[filter-network]: https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/
[hli-dr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[hli-backup]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#backup-and-restore
[hli-hadr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[hli-infrastructure]: /azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity
[hli-overview]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[hli-troubleshoot]: /azure/virtual-machines/workloads/sap/troubleshooting-monitoring
[ip]: https://blogs.msdn.microsoft.com/saponsqlserver/2018/02/10/setting-up-hana-system-replication-on-azure-hana-large-instances/
[network-best-practices]: /azure/security/azure-security-network-security-best-practices
[nfs]: /azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[os-hardening]: /azure/security/azure-security-iaas
[physical]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[planning]: /azure/vpn-gateway/vpn-gateway-plan-design
[protecting-sap]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/06/protecting-sap-systems-running-on-vmware-with-azure-site-recovery/
[ref-arch]: /azure/architecture/reference-architectures/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[region]: https://azure.microsoft.com/global-infrastructure/services/
[running-sap-blog]: https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/
[quick-sizer]: https://service.sap.com/quicksizing
[sap-1793345]: https://launchpad.support.sap.com/#/notes/1793345
[sap-1872170]: https://launchpad.support.sap.com/#/notes/1872170
[sap-2121330]: https://launchpad.support.sap.com/#/notes/2121330
[sap-2159014]: https://launchpad.support.sap.com/#/notes/2159014
[sap-1736976]: https://launchpad.support.sap.com/#/notes/1736976
[sap-2296290]: https://launchpad.support.sap.com/#/notes/2296290
[sap-community]: https://www.sap.com/community.html
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[scripts]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[sku]: /azure/expressroute/expressroute-about-virtual-network-gateways
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[stack-overflow]: https://stackoverflow.com/tags/sap/info
[stonith]: /azure/virtual-machines/workloads/sap/ha-setup-with-stonith
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[type]: /azure/virtual-machines/workloads/sap/hana-installation
[vnet]: /azure/virtual-network/virtual-networks-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx