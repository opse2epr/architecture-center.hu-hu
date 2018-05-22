---
title: Azure-példányokon nagy SAP HANA futtatása
description: Eljárások az SAP HANA magas rendelkezésre állású környezetben futó Azure nagy példányok bizonyítása.
author: lbrader
ms.date: 05/16/2018
ms.openlocfilehash: 7605fa8a0012aaef3f7323c6f88614b640152e3b
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/21/2018
---
# <a name="run-sap-hana-on-azure-large-instances"></a>Azure-példányokon nagy SAP HANA futtatása

A referencia-architektúrában egy SAP HANA működő (nagy példányok) Azure-magas rendelkezésre állású és vész-helyreállítási eljárásai bevált készletét jeleníti meg. HANA nagy példányok neve, az ajánlatot a telepített fizikai kiszolgálók Azure-régiók. 

![0][0]

> [!NOTE]
> A referenciaarchitektúra üzembe helyezéséhez SAP-termékek és más, nem a Microsoft által gyártott termékek megfelelő licence szükséges.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő infrastruktúra összetevőkből áll.

- **Virtuális hálózat**. A [Azure Virtual Network] [ vnet] szolgáltatás biztonságosan Azure-erőforrások kapcsolódik egymáshoz, és külön oszlik [alhálózatok] [ subnet] az egyes rétegekhez. SAP alkalmazás rétegek telepített Azure csúcsszintű gépek (VM) a nagy példányok levő HANA Adatbázisréteg való kapcsolódáshoz.

- **Virtuális gépek**. Virtuális gépek szerepelnek a SAP alkalmazásréteg és a megosztott szolgáltatások réteg. Ez utóbbi tartalmazza a rendszergazdák által használt HANA nagy példányok és más virtuális gépek eléréséhez jumpbox. 

- **HANA nagy példány**. A [fizikai kiszolgáló] [ physical] hitelesített SAP HANA igazított Datacenter integrációs (TDI) szabványainak való megfelelés érdekében futtatja az SAP HANA. Ez az architektúra két nagy HANA-példányt használ: egy elsődleges és másodlagos számítási egység. Magas rendelkezésre álláshoz az adatok rétegben HANA rendszer replikációs (HSR) keresztül valósul meg.

- **Magas rendelkezésre állású pár**. HANA nagy példányok paneleken csoportja felügyelete együtt, hogy alkalmazás redundanciát és megbízhatóságát. 

- **(A Microsoft vállalati peremhálózati) MSEE**. MSEE egy kapcsolat szolgáltatójánál vagy a hálózati biztonsági ExpressRoute-kapcsolatcsoportot keresztül kapcsolódási pontot. 

- **A hálózati adapterek (NIC)**. Ahhoz, hogy a kommunikációs, a nagy HANA-példány kiszolgálójának alapértelmezés szerint négy virtuális hálózati adapter biztosít. Ez az architektúra ügyfél-kommunikációt, a második hálózati adapter HSR, egy külső hálózati adapter HANA nagy példány tárolására és az iSCSI-fürtszolgáltatás magas rendelkezésre állású szerepel egy negyedik által igényelt csomópont-csomópont csatlakoztatásához egyetlen hálózati Adapterrel kell rendelkeznie.
    
- **Hálózati fájlrendszer (NFS) tárolás**. A [NFS] [ nfs] kiszolgáló támogatja a védett adatok megőrzése HANA nagy példány biztosító hálózati fájlmegosztáson.

- **ExpressRoute.** [ExpressRoute] [ expressroute] magánhálózati kapcsolatok egy a helyszíni hálózat és ne lépje át a nyilvános internethez Azure virtuális hálózatok közötti létrehozásának ajánlott Azure hálózati szolgáltatás. Azure virtuális gépek csatlakozni egy másik ExpressRoute-kapcsolatot használó HANA nagy példányokat. Az Azure virtuális hálózat és a HANA nagy példányok között ExpressRoute-kapcsolat a Microsoft ajánlat részeként van beállítva.

- **Átjáró**. Az ExpressRoute-átjárót az Azure virtuális hálózat az SAP alkalmazásréteg HANA nagy példány hálózati használt csatlakozáshoz használt. Használja a [nagy teljesítményt és Ultra teljesítmény] [ sku] Termékváltozat.

- **Vészhelyreállítás (DR)**. Kérésre tárolóreplikálást támogatott, és konfigurálja az elsődleges helyről a [vész-Helyreállítási hely] [ DR-site] egy másik régióban található.  
 
## <a name="recommendations"></a>Javaslatok
Követelmények is eltérőek, ezért ezek a javaslatok kiindulási pontként.

### <a name="hana-large-instances-compute"></a>Nagy HANA-példányok számítási
[Nagy példányok] [ physical] fizikai kiszolgálók az Intel EX E7 CPU architektúra alapján, és egy nagy példány stamp beállítva – ez azt jelenti, hogy egy adott készletét kiszolgálók vagy a paneleken. A számítási egység eredménye egy kiszolgáló vagy panelt, és egy stamp tevődik össze több kiszolgálóhoz vagy paneleken. Egy nagy példány stamp belül a kiszolgálók nem megosztott, és az SAP HANA egy ügyfél-telepítést futtató számára vannak fenntartva.

Az SKU különböző HANA nagy példányok, 20 TB egyetlen példányát (60 TB kibővített) memória az S/4HANA vagy más SAP HANA-munkaterhelések támogató érhetők el. [Két osztály] [ classes] kiszolgálók érhető el:

- I. osztály típusa: S72, S72m, S144, S144m, S192 és S192m

- Írja be a II osztály: S384, S384m, S384xm, S576m, S768m és S960m

Például a S72 SKU 768 GB RAM, tárolási és 2 Intel Xeon processzor (E7-8890 v3) 36 magok 3 terabájt (TB) tartalmaz. Válassza ki a Termékváltozat a felépítéséről és kialakításáról munkamenetekben meghatározott méretezési követelményeinek. Mindig győződjön meg arról, hogy a méretezési vonatkozik-e a megfelelő Termékváltozat. Képességek és a telepítési követelményeket [típus szerint eltérőek][type], és függ a rendelkezésre állási [régió][region]. Akkor is is fokozzák a egyik Termékváltozatáról egy nagyobb másikra.

A Microsoft segítségével, a nagy példányok beállítása rögzíthető, de a felelőssége, hogy az operációs rendszer konfigurációs beállításainak ellenőrzése. Mindenképpen tekintse át a legfrissebb SAP pontos Linux kiadási megjegyzéseket.

### <a name="storage"></a>Storage
A tárolási elrendezés SAP Hana a TDI ajánlása alapján van megvalósítva. Olyan speciális tárolási konfigurációt a szabványos TDI specifikációk HANA nagy példányok rendelkeznek. 1 TB-os lépésekben a kiegészítő tárhelyet is vásárolhat. 

A kritikus fontosságú környezeteket, a gyors helyreállítás követelményeinek kielégítése érdekében NFS használatban és nem közvetlenül csatlakoztatott tárolók. Az NFS-tároló kiszolgáló HANA nagy példányok több-bérlős környezetben, ahol bérlők különítik el és számítási, hálózati és tárolóerőforrás-elkülönítés használatával biztonságossá tárolja.

Támogatja a magas rendelkezésre állást, az elsődleges helyen, használjon másik tárolási elrendezést. A több gazdagépen kibővített, például a tárolási van osztva. Egy másik magas rendelkezésre állású lehetőség például HSR alkalmazás-alapú replikálás. Dr, azonban a pillanatkép-alapú tárolási replikálását alkalmazza.

### <a name="networking"></a>Hálózat
Ez az architektúra és a fizikai, mind a virtuális hálózatokat használ. A virtuális hálózat Azure IaaS része, és egy különálló HANA nagy példányok fizikai hálózaton keresztül kapcsolódik [ExpressRoute][expressroute]] kapcsolatok. A létesítmények közötti átjáró csatlakozik a helyszíni helyeken a munkaterhelés Azure virtuális hálózatban.

HANA nagy példányok hálózatok különítve egymástól a biztonság. Különböző régiókban található példányok nem kommunikálhatnak egymással, a dedikált tárolási replikációs kivételével. Azonban a HSR használatához régió közötti kommunikáció szükség. [IP-útválasztási táblázataiba] [ ip] vagy proxyk használható kereszt-régiók HSR engedélyezéséhez.

Minden Azure virtuális hálózatokhoz csatlakozó HANA nagy példányok egy régióban lehet [határokon csatlakozó] [ cross-connected] HANA nagy példányokhoz egy másodlagos régióban ExpressRoute keresztül.

ExpressRoute HANA nagy példányok kiépítése során alapértelmezés szerint tartalmazza. A telepítés csak bizonyos hálózati elrendezése van szükség, többek között a szükséges CIDR-címtartományok és az útválasztási tartományhoz. További információkért lásd: [SAP HANA (nagy példányok) infrastruktúra és az Azure-kapcsolatokat][HLI-infrastructure].

## <a name="scalability-considerations"></a>Méretezési szempontok
Felfelé vagy lefelé méretezési, választhat a kiszolgálók által biztosított HANA nagy példányok sok méretét. Azok ki vannak üzletiként besorolva [i. típusú és II] [ classes] és igazodjon a különböző munkaterhelések. Válassza ki a méretét, a számítási feladatok a következő három évben. Egy éves kötelezettségvállalások is elérhetők.

Egy több gazdagép kibővített telepítése általában adatbázis particionálási stratégia egyfajta használt BW/4HANA telepítésekhez. Horizontális, tervezze meg a telepítés előtti HANA táblák elhelyezését. Az infrastruktúra szempontból több gazdagép csatlakozik megosztott tárolókötethez abban az esetben, ha a számítási munkavégző csomópontok az HANA rendszerben egyike meghibásodik, amely lehetővé teszi, hogy gyors felvásárlási készenléti állomások.

S/4HANA és az SAP Business Suite HANA egy egyetlen panelen a méretezhető legfeljebb 20 TB-os HANA nagy példányok egypéldányos.

Greenfield forgatókönyvek esetén a [SAP gyors Szimbólumméretező] [ quick-sizer] SAP szoftver fölött HANA végrehajtásának memóriaigényének kiszámításához érhető el. Memóriára vonatkozó követelményeknek Hana növelje az adatmennyiség növekedésével. A rendszer aktuális memória-felhasználás alapjaként használni a későbbi felhasználás előrejelzéséhez, és képezze le az igény szerinti valamelyikébe a HANA nagy példányok méretű.

Ha már rendelkezik SAP központi telepítések, a SAP biztosít jelentések segítségével ellenőrizze a meglévő rendszerek által használt adatokat, és számítja ki a memória a HANA példánya. Például tekintse meg a következő SAP megjegyzéseket: 

- SAP Megjegyzés [1793345] [ sap-1793345] -HANA az SAP Suite méretezése
- SAP Megjegyzés [1872170] [ sap-1872170] -csomag HANA és S/4 HANA méretezési jelentésre
- SAP Megjegyzés [2121330] [ sap-2121330] -gyakran ismételt kérdések: a jelentés méretezése HANA SAP BW Programhoz
- SAP Megjegyzés [1736976] [ sap-1736976] -HANA BW jelentés méretezése
- SAP Megjegyzés [2296290] [ sap-2296290] – új jelentés HANA BW méretezése

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Erőforrás redundanciát a magas rendelkezésre állású infrastruktúramegoldásainak általános téma. A vállalatok számára, amelyek kevésbé szigorú SLA-t egypéldányos Azure virtuális gépek nyújtanak hasznos üzemidőt SLA-t. További információkért lásd: [Azure szolgáltatásiszint-megállapodás](https://azure.microsoft.com/support/legal/sla/).

SAP, a rendszer integráló, vagy a Microsoft megfelelően tervezővel és valósíthatnak meg dolgozni egy [magas rendelkezésre állású és vész-helyreállítási] [ hli-hadr] stratégia. Ez az architektúra követi az Azure [szolgáltatásiszint-megállapodás] [ sla] (SLA) Hana Azure (nagy példány). A rendelkezésre állási követelményeinek felmérése, fontolja meg a hiba, a szolgáltatások hasznos üzemidő kívánt szintje és a közös metrikákat a hibaérzékeny pontokat:

- A helyreállítási idő célkitűzés (RTO) azt jelenti, hogy az az időtartam, amelyen a HANA nagy példányok kiszolgáló nem érhető el.

- Helyreállítási pont célkitűzés (RPO) azt jelenti, hogy a maximális megengedhető idő, mely az ügyfelek tárolt adatok elvesztésével járhat egy hiba miatt.

A magas rendelkezésre állás érdekében telepíteni a magas rendelkezésre ÁLLÁSÚ párban egynél több példány, és segítségével HSR szinkron módban az adatvesztéssel és az állásidő minimalizálása érdekében. Egy helyi, két csomópontos magas rendelkezésre állású összeállításban mellett HSR támogatja a többrétegű replikációját, harmadik csomópont egy külön Azure-régió, ahol a fürtözött HSR párjával replikációs célhelye másodlagos replikája regisztrálása. Ez képezi egy replikációs lánckapcsolású láncot. A feladatátvételt a vész-Helyreállítási csomópontra manuális folyamatban.

Beállításakor HANA nagy példányok HSR automatikus feladatátvétellel, kérhet a Microsoft szolgáltatás-felügyeleti csoport nem állítson be egy [STONITH eszköz] [ stonith] a meglévő kiszolgálói számára. 

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok
Ez az architektúra támogatja az [vész-helyreállítási] [ hli-dr] HANA-különböző Azure-régiók nagy példánya között. DR HANA nagy osztályt támogatásához két módja van:

- Tárolóreplikálást. Elsődleges tárolási tartalma folyamatosan replikálva vannak a távoli vész-Helyreállítási tárolórendszerek, amelyek a kijelölt vész-Helyreállítási HANA nagy példányok kiszolgálón érhetők el. A memóriába nincs betöltve a storage replikáció esetén a HANA-adatbázisból. A vész-Helyreállítási lehetőség egyszerűbb felügyeleti szempontból. Annak megállapítása, hogy ez megfelelő stratégiát, fontolja meg az adatbázis betöltési ideje az SLA-elérhetőséget ellen. Tárolási replikációs is lehetővé teszi időpontban helyreállításhoz. Többfunkciós (költség optimalizált) vész-Helyreállítási be van állítva, további tárhely a vész-Helyreállítási helyen azonos méretűnek kell vásárolnia. A Microsoft biztosít self-services [tárolási pillanatkép és feladatátvételi parancsfájlok] [ scripts] HANA feladatátvételi az HANA nagy példányok elérhető részeként.

- Többrétegű HSR a vész-Helyreállítási régióban (ahol a HANA-adatbázisból be van töltve, a memória) egy harmadik replikával. Ezt a beállítást a gyorsabb helyreállítás támogatja, de nem támogatja a időpontban helyreállítást. HSR másodlagos rendszer igényel. A vész-Helyreállítási hely HANA replikációs például nginx vagy IP-táblák proxyk segítségével kezel. 

> [!NOTE]
> A referencia-architektúrában költségek egypéldányos környezetben futó is optimalizálhatja. Ez [költség optimalizált forgatókönyv](https://blogs.sap.com/2016/07/19/new-whitepaper-for-high-availability-for-sap-hana-cost-optimized-scenario/) nem éles HANA munkaterhelések alkalmas. 

## <a name="backup-considerations"></a>Biztonsági másolat szempontjai
Több lehetőség érhető el az üzleti követelmények alapján választhat [biztonsági mentési és helyreállítási][hli-backup].

| Ez a beállítás                   | Informatikai szakemberek                                                                                                   | Hátrányok                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| HANA biztonsági mentése        | SAP natív. Beépített konzisztencia-ellenőrzést.                                                             | Biztonsági mentési és helyreállítási folyamata hosszú. Tárolási lemezterület-felhasználást. |
| HANA pillanatkép      | SAP natív. Gyors mentését és helyreállítását.                                                               |                                       |
| Tárolási pillanatkép   | HANA nagy osztályt tartalmazza. Optimalizált vész-Helyreállítási HANA nagy példányok. Rendszerindító kötet mentést készíthet. | Maximális 254 pillanatképek kötetenként.                          |
| Napló biztonsági mentése         | Az időpontra történő visszaállítás szükséges.                                                                   |                                                            |
| Egyéb biztonsági mentési eszközök | Redundáns biztonsági mentési helyre.                                                                             | További licencelési költségeit.                                |

SapHanaTutorial.com emellett egy hasznos cikk [HANA biztonsági mentési lehetőségek összehasonlítása][sap-hana-tutorial].

## <a name="manageability-considerations"></a>Felügyeleti szempontok
HANA nagy példányok erőforrások, például a Processzor, memória, hálózati sávszélességet és tárolóhelyet, a SAP HANA Studio, a SAP HANA-Vezérlőpult, a SAP megoldás Manager és az egyéb natív Linux eszközök figyelésére. HANA nagy példányok beépített Hálózatfigyelő eszközök nem rendelkeznek. A Microsoft forrásanyagokat biztosít az [elhárításukkal és a figyelheti] [ hli-troubleshoot] a szervezet követelményeinek, és a Microsoft támogatási szolgálatához megfelelően csapatunk segítségére a műszaki problémák elhárításához. 

Ha további számítási funkció, ha előbb telepítik azokra a nagyobb Termékváltozat. 

## <a name="security-considerations"></a>Biztonsági szempontok
- Alapértelmezés szerint HANA nagy példányok tárolás titkosítása a TDE (átlátható adattitkosítás) az adatok aktívan használnak.

- Nagy HANA-példányok és a virtuális gépek közötti átvitel adatok nem titkosítottak. Az adatátvitel titkosításához, az alkalmazás-specifikus titkosítás engedélyezése. Lásd az SAP megjegyzést [2159014] [ sap-2159014] -gyakran ismételt kérdések: SAP HANA biztonsági.

- Elkülönítés a több-bérlős HANA nagy példány környezetben a bérlők közötti biztonsági biztosít. Bérlők el választva egymástól a saját VLAN használatával.

- [Azure-hálózat ajánlott biztonsági eljárások] [ network-best-practices] hasznos útmutatást nyújtanak.

- Csakúgy, mint a központi telepítési, [operációs rendszer korlátozásának] [ os-hardening] ajánlott.

- A fizikai biztonság Azure adatközpontjaiban való hozzáférés csak az arra jogosult személyek korlátozva. Nincs ügyfelek is hozzáférhetnek a fizikai kiszolgálók.

További információkért lásd: [SAP HANA biztonsági – áttekintés egy][sap-security]. () Egy SAP szolgáltatás piactér fiókot kell megadni a hozzáférést.)

## <a name="communities"></a>Közösségek
A közösségek választ adhatnak a kérdéseire, továbbá segíthetnek a sikeres üzembe helyezésben. A következőket ajánljuk figyelmébe:

* [A Microsoft Platform blog futó SAP-alkalmazásokból][running-sap-blog]
* [Az Azure közösségi támogatás][azure-forum]
* [SAP-Közösség][sap-community]
* [SAP veremtúlcsordulás][stack-overflow]

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
[quick-sizer]: http://service.sap.com/quicksizing
[sap-1793345]: https://launchpad.support.sap.com/#/notes/1793345
[sap-1872170]: https://launchpad.support.sap.com/#/notes/1872170
[sap-2121330]: https://launchpad.support.sap.com/#/notes/2121330
[sap-2159014]: https://launchpad.support.sap.com/#/notes/2159014
[sap-1736976]: https://launchpad.support.sap.com/#/notes/1736976
[sap-2296290]: https://launchpad.support.sap.com/#/notes/2296290
[sap-community]: https://www.sap.com/community.html
[sap-hana-tutorial]: http://saphanatutorial.com/comparison-between-hana-backup-options/
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[scripts]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[sku]: /azure/expressroute/expressroute-about-virtual-network-gateways
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[stack-overflow]: http://stackoverflow.com/tags/sap/info
[stonith]: /azure/virtual-machines/workloads/sap/ha-setup-with-stonith
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[type]: /azure/virtual-machines/workloads/sap/hana-installation
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/sap-hana-large-instances.png "Azure nagy példányok használatával SAP HANA-architektúra"
