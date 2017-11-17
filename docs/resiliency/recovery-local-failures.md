---
title: "Műszaki útmutató: az Azure-ban helyi hibaelhárítás"
description: "Szóló ismertetése és rugalmas, magas rendelkezésre állású kialakítása, hibatűrő alkalmazások, valamint az Azure helyi hibák összpontosított vészhelyreállítás tervezése."
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 180eb465e5f82406bb03924a29d5b06d43bbaa24
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Az Azure rugalmasságát műszaki útmutatót: helyi hibaelhárítás az Azure-ban

Számos alkalmazás rendelkezésre állásának két elsődleges fenyegetéseket.

* Eszközök, például lemezek vagy kiszolgálók hibája
* A kritikus erőforrásokat, például a számítási munkaterhelés esetére csúcs kimerülése

Azure biztosít erőforrás-kezelést, a rugalmasság, terheléselosztás és ilyen körülmények magas rendelkezésre állásának biztosítása particionálás kombinációja. A felsorolt szolgáltatások némelyikéhez automatikusan megtörténik az összes Azure-szolgáltatásokhoz. Egyes esetekben azonban az alkalmazás fejlesztőjének tegye őket kihasználják néhány további feladata.

## <a name="cloud-services"></a>Cloud Services
Azure Cloud Services áll legalább egy webes vagy feldolgozói szerepköröket gyűjteményei. Egy vagy több példánya egy egyidejűleg is futtathatók. A konfiguráció a példányok száma határozza meg. Szerepkörpéldányokat monitorozott és felügyelt keresztül a fabric controller összetevőt. A fabric controller észleli, és válaszol-e a szoftver- és hardverkonfigurációiról hibák automatikusan.

Minden szerepkörpéldányt a saját virtuális gép (VM) fut, és a háló ellenőrzi a vendégügynök keresztül kommunikál. A vendégügynök erőforrás és a csomópont metrika, beleértve a virtuális gép használati, állapota, naplók, erőforrás-használat, kivételek és meghibásodás gyűjti. A fabric controller lekérdezi a vendégügynök konfigurálható időközönként, és azt a virtuális gép újraindul, ha a vendégügynök nem válaszol. Hardverhiba esetén a társított fabric controller minden érintett szerepkörpéldányokat áthelyezése új hardver csomópontot, és újrakonfigurálja a hálózati forgalom van.

Is kihasználhatják ezeket a funkciókat, a fejlesztők győződjön meg arról, hogy az összes szolgáltatás szerepkör elkerülése érdekében a szerepkörpéldányok állapotát tárolja. Ehelyett minden állandó adatok legyenek elérhetők a tartós tárból, például az Azure Storage vagy az Azure SQL Database. Ez lehetővé teszi a tanúsítványigénylések szerepköröket. Azt is jelenti, hogy szerepkörpéldányokat is leáll bármikor inkonzisztenciát létrehozása a szolgáltatás átmeneti vagy állandó szerinti nélkül.

A követelmény a szerepkörök a külsőleg állapot tárolására több hatással van. Ez azt jelenti, például, hogy minden kapcsolódó módosítások Azure Storage táblához úgy kell módosítani entitás-csoport egyetlen tranzakcióban, ha lehetséges. Természetesen nem mindig minden változtatásokat egy tranzakción belül. Győződjön meg arról, hogy szerepkör példány hibák nem problémákat okozhat, ha azok megszakítási hosszú futású műveleteket, amelyek a szolgáltatás állandó állapotát két vagy több frissítést több különös gondot kell végrehajtania. Ha egy másik szerepkört, majd ismételje meg az ilyen művelet hajt végre, azt kell várhatóan, és kezeli az esetet, ahol a munkahelyi részben fejeződött be.

Vegye figyelembe például olyan szolgáltatás, amely adatok partíciók több üzletben között. Ha a feldolgozói szerepkör leáll, amíg azt van áthelyezése egy shard, valószínűleg nem fejeződött be a shard történő áttelepítését. Vagy az adatáthelyezési előfordulhat, hogy ismétlődik a kezdetektől a különböző feldolgozói szerepkör, potenciálisan a árva adatok vagy adatsérülés okozza. Problémák megelőzése érdekében hosszú futású műveleteket kell lennie a következők közül:

* *Az Idempotent*: ismételhető hatásai nélkül. Az idempotent, a hosszan futó műveletet kell rendelkeznie ugyanaz a hatása, függetlenül attól, hogy hányszor program hajtsa végre, akkor is, ha a végrehajtás során megszakad.
* *Növekményesen újraindítható*: a legutóbbi pont a hiba továbbra is. Kisebb atomi műveletek sorozata lehet Növekményesen újraindítható-e, hogy a hosszan futó műveletet kell állnia. Azt is rögzíteni kell a telepítés előrehaladását a tartós tároló, hogy minden ezt követő meghívása átveszi ahol annak elődje leállt.

Végül minden hosszan futó műveletet kell hívható meg ismételten amíg nem sikerül. Például a telepítési művelet lehet, hogy kell helyezni egy Azure-üzenetsorba, és eltávolította a várólista által a feldolgozói szerepkör csak akkor, ha ez sikeres. Lehet, hogy a szemétgyűjtő létrehozásához szükséges adatok, amelyek műveletek megszakadt karbantartása.

### <a name="elasticity"></a>A rugalmasság
A kezdeti száma az egyes szerepkörökhöz futó példányok minden egyes szerepkör-konfigurációja határozza meg. Rendszergazdák minden egyes szerepkör futtatásához két vagy több osztályt a tervezett terhelés alapján állít be kell beállítani. De könnyen költenie szerepkörpéldányok, vagy le, a használati minták módosítása. Ehhez manuálisan az Azure portálon, vagy a Windows PowerShell, a Service Management API vagy a külső eszközök segítségével automatizálható a folyamat. További információkért lásd: [automatikus skálázás alkalmazás hogyan](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Particionálás
Az Azure fabric controller partíciók két típusú használ:

* Egy *frissítési tartomány* frissíteni az adott szolgáltatás szerepkörpéldányt csoportok használatával. Azure több frissítési tartományt a szolgáltatáspéldány telepíti. Helybeni frissítést a háló tartományvezérlő összes példányát le egy frissítési tartomány számos lehetőséget kínál, frissítése és majd újraindítja a következő frissítési tartományra áthelyezése előtt. Ez a megközelítés megakadályozza a teljes szolgáltatás nem érhető el a frissítési folyamat alatt.
* A *tartalék tartomány* határozza meg a hardver- vagy hálózati lehetséges pontokat. Olyan szerepkört, amely egynél több példány van a fabric controller biztosítja, hogy a példányok több tartalék tartományok, ezáltal megakadályozhatja, hogy a elkülönített hardver meghibásodása megzavarása különböző pontjain. Tartalék tartományok összes lehetősége, hogy a kiszolgáló és fürt hibák tekintetében.

A [Azure szolgáltatásiszint-szerződés (SLA)](https://azure.microsoft.com/support/legal/sla/) garantálja, hogy ha két vagy több webalkalmazás szerepkörpéldányokat különböző tartalék és verziófrissítési van telepítve, akkor kell külső kapcsolatot legalább 99,95 %-ában. Ellentétben frissítési tartományok nincs lehetőség a tartalék tartományok száma. Azure automatikusan lefoglal tartalék tartományok, és a szerepkörpéldányok elosztja. At legalább az első két példánya minden kerülnek, különböző tartalék és frissítési tartományt annak érdekében, hogy legalább két példánnyal szerepköröket eleget tesz a szolgáltatásiszint-szerződést. Ez a következő ábra a jelzi.

![Tartalék tartomány elkülönítési egyszerűsített nézete](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Terheléselosztás
Az összes bejövő forgalmát a webes szerepkör haladnak keresztül az állapot nélküli terheléselosztó, amely továbbítja az ügyfélkéréseket a szerepkörpéldányok között. Az egyes szerepkörpéldányokat nem rendelkezik nyilvános IP-címeket, és nincsenek közvetlenül címezhető az internetről. Webes szerepkörök olyan állapot nélküli, így egyetlen ügyfélkérés bármely szerepkör példánya lehet továbbítani. A [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) esemény 15 másodpercenként jelenik meg. Ezzel azt jelzi, hogy a szerepkör készen áll a forgalom fogadására, vagy hogy-e elfoglalva más műveletekkel, és a terheléselosztó Elforgatás kívül kell venni.

## <a name="virtual-machines"></a>Virtuális gépek
Azure virtuális gépek platform eltér, a szolgáltatás (PaaS) számítási szerepkörök számos tekintetben magas rendelkezésre állású viszonyítva. Bizonyos esetekben a magas rendelkezésre állásának biztosításához a további munkahelyi tegye.

### <a name="disk-durability"></a>Lemez tartóssági
PaaS szerepkörpéldányok, ellentétben a virtuális gép meghajtókon tárolt adatokat az állandó akkor is, ha a virtuális gép áthelyezését van. Az Azure virtuális gépek használata az Azure Storage blobs létező virtuális gépek lemezei. Azure Storage rendelkezésre állási jellemzői miatt a virtuális gép meghajtókon tárolt adatokat is magas rendelkezésre áll.

Vegye figyelembe, hogy (a Windows-alapú virtuális gépek) D meghajtó az ezen szabály alól. D meghajtó ténylegesen fizikai tároló a állvány kiszolgálón, amelyen a virtuális Gépet, és az adatok elvesznek, ha a virtuális gép újraindul. D meghajtó csak az átmeneti tárolási számára készült. A Linux Azure "általában" (de nem minden esetben) mutatja a helyi ideiglenes lemez /dev/sdb elérésű eszközt. Gyakran csatlakoztatva van az Azure Linux ügynök /mnt/resource vagy /mnt csatlakoztatási pontokat (/etc/waagent.conf állíthatók be).

### <a name="partitioning"></a>Particionálás
Azure natív módon megértette a rétegek egy PaaS-alkalmazás (webes és feldolgozói szerepkörben), és így tudja megfelelően kioszthatja hiba és a frissítési tartományok között. Ezzel szemben a rétegek szolgáltatási (IaaS) alkalmazás infrastruktúrákban manuálisan definiálni kell keresztül a rendelkezésre állási készletek. Szolgáltatásiszint-szerződésben garantált IaaS a rendelkezésre állási készletek szükségesek.

![Az Azure virtuális gépek rendelkezésre állási készletek](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

A fenti ábrán az Internet Information Services (IIS) réteg (amely működik, mint a web app réteget) és az SQL-réteg (amely egy adatrétegbeli működik) vannak hozzárendelve másik rendelkezésre állási készletek. Ez biztosítja, hogy az egyes rétegek összes példányát tartalék tartományok közötti virtuális gépek elosztásával hardver redundanciával rendelkezik, és hogy teljes rétegek nem veszik frissítése közben.

### <a name="load-balancing"></a>Terheléselosztás
A virtuális gépek elosztott ezek között forgalmat kell rendelkeznie, ha egy alkalmazás és a terhelés elosztása a virtuális gépek között egy adott TCP vagy UDP-végpontot kell csoportosítja. További információkért lásd: [terheléselosztási virtuális gépek](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). A virtuális gépek más forrásból (például egy üzenetsor-kezelési mechanizmust) fogadja, ha egy terhelés-kiegyenlítő nincs szükség. A load balancer egy egyszerű állapot-ellenőrzést annak meghatározásához, hogy forgalmat kell küldeni a csomópont használja. Akkor is valósítja meg az alkalmazás-specifikus állapotfigyelő metrikákat, amelyek meghatározzák, hogy a VM forgalmat kell kapnia a saját mintavételt létrehozásához.

## <a name="storage"></a>Storage
Az Azure Storage egy az eredeti tartós szolgáltatás az Azure-bA. Blob, table, várólista és VM lemezegységet biztosít. Magas rendelkezésre állás egyetlen adatközponton belül replikációs és erőforrás-kezelést használ. Az Azure Storage SLA-elérhetőséget biztosítja azt, hogy legalább 99,9 %-ában:

* Helytelenül formázott kérelmek hozzáadása, update, Olvasás és az adatok sikeresen és helyesen dolgozza fel a rendszer.
* Storage-fiókok és az internetes átjáró lesz.

### <a name="replication"></a>Replikáció
Az Azure Storage adatok tartóssága könnyíti meg a különböző meghajtókon összes adatok többszörös lemásolását, megőrizve a teljesen független fizikai tároló alrendszerek abban a régióban. Adatait replikálja a rendszer szinkron módon történik, és minden másolatot véglegesítése előtt elfogadott, az írás. Az Azure Storage minden erősen konzisztens, tehát, hogy olvasási garantáltan tükrözi a legfrissebb írások. Ezenkívül adatok másolatát folyamatosan felismerése és kijavítása bit rothadás, egy gyakran overlooked fenyegetést tárolt adatok vizsgálata.

Azure Storage használatával csak replikációs szolgáltatások hasznos. A service fejlesztői helyreállítás egy helyi történt hiba után további munkájuk elvégzéséhez nem szükséges.

### <a name="resource-management"></a>Erőforrás-kezelés
Tárfiókok létrehozása után előfordulhat, hogy 2014, milyen mértékben növelhető legfeljebb 500 TB kapacitású lehet (a korábbi maximális volt 200 TB). További lemezterületre akkor szükség, ha alkalmazásokat kell megtervezni, több tárfiókot használja.

### <a name="virtual-machine-disks"></a>Virtuális gépek lemezei
A virtuális gép lemezét oldalakra vonatkozó blob az Azure Storage adjon neki ugyanazokat tartósság és a méretezhetőség tulajdonságai, a Blob Storage tárolóban tárolja. Ez a kialakítás teszi az adatok egy virtuális gép lemezét állandó, még akkor is, ha a virtuális Gépet futtató kiszolgáló meghibásodik, és újra kell indítani a virtuális gép egy másik kiszolgálón.

## <a name="database"></a>Adatbázis
### <a name="sql-database"></a>SQL Database
Az Azure SQL Database adatbázist biztosít szolgáltatásként. Alkalmazások gyors kiépítése, adatok, és a lekérdezés relációs adatbázisok beszúrása lehetővé teszi. Számos, a megszokott SQL Server szolgáltatásai és funkciói, absztrakt módon megjelenítve a hardver, a konfiguráció, a javítás és a rugalmasság okozta terheket közben biztosít.

> [!NOTE]
> Az Azure SQL-adatbázis nem biztosít az SQL Server-az-egyhez szolgáltatásparitást. Az különböző bizonyos követelmények – egy egyedi módon van igazodó felhőalkalmazásokhoz (rugalmasan méretezhető, adatbázis karbantartási költségek csökkentése, és így tovább) teljesítéséhez rendelkezik szolgál. További információkért lásd: [felhőalapú SQL Server-verzió: Azure SQL (PaaS) Database vagy az SQL Server Azure virtuális gépeken (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).
> 
> 

#### <a name="replication"></a>Replikáció
Az Azure SQL Database csomópont szintű hiba beépített rugalmasságot biztosít. Két vagy több háttér csomópont egy kvórum véglegesítési technika keresztül automatikusan replikált minden írás az adatbázisba. (Az elsődleges és legalább egy másodlagos meg kell erősítenie, hogy a tevékenység ír a tranzakciós napló előtt a tranzakció akkor számít elértnek sikeres, és adja vissza.) Csomópont meghibásodása esetén az adatbázis automatikusan átadja a feladatokat a másodlagos replikák közül. Ez azt eredményezi, hogy egy átmeneti kapcsolat megszakadása ügyfélalkalmazások esetében. Emiatt minden Azure SQL Database-ügyfelek kell megvalósítania valamilyen átmeneti kapcsolat kezelése. További információkért lásd: [ismételje meg a szolgáltatás konkrét útmutatást](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Erőforrás-kezelés
Minden adatbázis létrehozásakor, és a méretének felső korlátja van konfigurálva. A jelenleg rendelkezésre álló maximális mérete 1 TB-os (korlátok változó méretű lásd a szolgáltatási réteg alapján [szolgáltatás és az Azure SQL-adatbázisok teljesítményszintek](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Egy adatbázis találatok maximális felső méretét, elutasítja a további Beszúrás vagy frissítés parancsok. (Kérdez le, és az adatok törlése az továbbra is lehetséges.)

Egy adatbázis Azure SQL Database használja a háló-erőforrások kezeléséhez. Azonban a fabric controller helyett használja egy körgyűrűs topológia észlelésére. A fürt minden replika két szomszédok és feladata az észlelése, amikor azok. Ha egy replika leáll, szomszédjainak egy újrakonfigurálás ügynököt újra létre kell hoznia azt egy másik gépen indítható el. Győződjön meg arról, hogy logikai kiszolgáló túl sok erőforrás használata a gépen nem, vagy haladhatja meg a számítógép fizikai korlátok motor szabályozás megadott.

### <a name="elasticity"></a>A rugalmasság
Ha az alkalmazás több mint 1 TB-os adatbázis korlát, alkalmaznia kell a kibővített megközelítést. A horizontális az Azure SQL Database által manuálisan particionálás, más néven horizontális, több SQL-adatbázisok közötti adatforgalom. Ezt a módszert használja a kibővített Itt a lehetőség, amely közel lineáris költség növekedés elérése. Rugalmas növekedésére vagy az igény szerinti kapacitás növekményes költséggel igény szerint növelhető mert adatbázisok átlagos tényleges mérete alapján napi használt nem maximális mérete alapján számlázzuk.

## <a name="sql-server-on-virtual-machines"></a>SQL Server a Virtual Machines szolgáltatásban
Ha telepíti az SQL Server (2014 vagy újabb verzió) Azure virtuális gépeken, kihasználhatja az SQL Server a hagyományos rendelkezésre állási funkciók. A funkciók közé tartoznak az AlwaysOn rendelkezésre állási csoportok és az adatbázis-tükrözés. Ne feledje, hogy Azure virtuális gépeken, tárolási és hálózatkezelési egy helyszíni informatikai infrastruktúra nem virtualizált eltérő működési jellemzői. Egy magas rendelkezésre állás és a vészhelyreállítás (elérhető HA/DR) SQL Server megoldást az Azure-ban sikeres végrehajtása szükséges, hogy ezek a különbségek megismeréséhez, és a megoldás kialakításakor megtervezése.

### <a name="high-availability-nodes-in-an-availability-set"></a>Magas rendelkezésre állású csomópont a rendelkezésre állási csoportok
Bevezetésekor egy magas rendelkezésre állású megoldás az Azure-ban, a rendelkezésre állási csoportot az Azure-ban segítségével Helyezzen el a magas rendelkezésre állású csomópontok külön tartalék tartományok és frissítési tartományt. Ahhoz, hogy törölje a jelet, a rendelkezésre állási csoport egy Azure fogalom. Fontos, hogy ellenőrizze, hogy az adatbázisok valóban magas rendelkezésre állású, hogy az AlwaysOn rendelkezésre állási csoportok, az adatbázis-tükrözés vagy valami más használata nyújtanak a legjobb. Ha nem követi a legmegfelelőbb eljárást alkalmazza, előfordulhat, hamis azt feltételezi, hogy magas rendelkezésre áll-e a rendszer alatt. De a valóságban, a csomópontok összes sikertelen lehet egyidejűleg mert akkor fordulhat elő, az azonos tartalék tartományban Azure-régióban kell elhelyezni.

Ez a javaslat nincs alkalmazhatók a naplóküldésben. Vész-helyreállítási szolgáltatás ellenőrizze, hogy a kiszolgálók az Azure-régiók külön futnak. E régiók definícióját, külön tartalék tartományok.

Azure Cloud Services telepített virtuális gépek a klasszikus portálon keresztül lehet az azonos rendelkezésre állási készlet telepítenie kell őket ugyanabban a Felhőszolgáltatásban található. Azure Resource Manageren keresztül (az aktuális portálon) telepített virtuális gépek nem kell ezt a korlátozást. A klasszikus portál telepített virtuális gépek Azure-felhőszolgáltatásban, csak az ugyanabban a Felhőszolgáltatásban található csomópontok részt az azonos rendelkezésre állási csoport. Ezenkívül a felhőalapú szolgáltatások virtuális gépeken annak érdekében, hogy az IP-címek megőriznek szolgáltatásjavításnak után is az azonos virtuális hálózatban kell lennie. Ezzel elkerülhető, hogy a DNS frissítési üzemzavarokhoz vezethet.

### <a name="azure-only-high-availability-solutions"></a>Csak Azure: magas rendelkezésre állású megoldások
Egy magas rendelkezésre állású megoldás az SQL Server-adatbázisokat az Azure-ban az AlwaysOn rendelkezésre állási csoportok vagy az adatbázis-tükrözés rendelkezik.

A következő ábra azt mutatja be, az Azure virtuális gépeken futó AlwaysOn rendelkezésre állási csoportok architektúrája. Ez az ábra a témával, a részletes cikkből készült [magas rendelkezésre állás és katasztrófa-helyreállítás az SQL Server Azure virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![AlwaysOn rendelkezésre állási csoportok a Microsoft Azure-ban](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Megadhat egy AlwaysOn rendelkezésre állási csoportok telepítési-végpontok Azure virtuális gépeken is automatikusan az Azure-portálon az AlwaysOn sablon használatával. További információkért lásd: [SQL Server AlwaysOn ajánlat a Microsoft Azure portál Gallery](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/).

A következő ábra bemutatja, hogy az adatbázis-tükrözés Azure virtuális gépeken. Részletes témakörben is készült [magas rendelkezésre állás és katasztrófa-helyreállítás az SQL Server Azure virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Adatbázis-tükrözés a Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> Mindkét architektúrát igényli tartományvezérlő. Azonban az adatbázis-tükrözés, lehetősége szükségtelenné teszik a tartományvezérlő kiszolgálói tanúsítványok segítségével.
> 
> 

## <a name="other-azure-platform-services"></a>Egyéb Azure platform szolgáltatásaiból
Azure készített alkalmazások számára előnyös platformképességet ahhoz, hogy a helyi hibák helyreállításához. Bizonyos esetekben az adott forgatókönyv rendelkezésre állásának növelése érdekében adott műveletek hajthatók végre.

### <a name="service-bus"></a>Service Bus
A veszély átmenetileg nem működik az Azure Service Bus, célszerű a tartós ügyféloldali várólista. Ez átmenetileg használja egy másik, helyi tárolási mechanizmus tárolja az üzeneteket, amelyek nem vehető fel a Service Bus-üzenetsorba. Az alkalmazás eldöntheti, hogyan legyen kezelve az ideiglenesen tárolt üzenetek, miután visszaállította a szolgáltatás. További információkért lásd: [gyakorlati tanácsok a teljesítménnyel kapcsolatos fejlesztések használatával a Service Bus közvetítőalapú üzenettovábbítás](/azure/service-bus-messaging/service-bus-performance-improvements/) és [Service Bus (katasztrófa utáni helyreállítás)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight
Az adatok Azure HDInsight társított alapértelmezett Azure Blob Storage tárolóban tárolja. Az Azure Storage Blob Storage magas rendelkezésre állású és a tartós tulajdonságokat adja meg. Hadoop-MapReduce-feladatok társított több csomópontos feldolgozásának a egy átmeneti Hadoop elosztott fájlrendszerrel (HDFS), amely ki van építve, ha a HDInsight az következik be. Eredményeit a MapReduce feladatot is tárolódnak az Azure Blob Storage tárolóban, alapértelmezés szerint, hogy a feldolgozott adatok tartósságát, és magas rendelkezésre állású marad, miután a Hadoop-fürt platformelőfizetés van. További információkért lásd: [HDInsight (katasztrófa utáni helyreállítás)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>A helyi hibák Ellenőrzőlisták
### <a name="cloud-services"></a>Cloud Services
1. Tekintse át a dokumentum a Felhőszolgáltatások szakaszát.
2. Konfigurálja az egyes szerepkörökhöz legalább két példányt.
3. Továbbra is fennáll a tartós tároló nem a szerepkörpéldányok állapotát.
4. A StatusCheck esemény megfelelően kezeli.
5. Tegye kapcsolódó módosítások tranzakciók, amikor lehetséges.
6. Ellenőrizze, hogy feldolgozói szerepkör feladatok idempotent és újraindítható folyamatok.
7. Továbbra is műveleteket hívhatnak meg, amíg nem sikerül.
8. Fontolja meg az automatikus skálázás stratégiák.

### <a name="virtual-machines"></a>Virtuális gépek
1. Tekintse át a virtuális gépek szakasz ebben a dokumentumban.
2. Ne használjon D meghajtó állandó tároló.
3. A szolgáltatási rétegben található gépek csoportosíthatja egy rendelkezésre állási csoportot.
4. Terheléselosztás és a választható mintavételt konfigurálása.

### <a name="storage"></a>Storage
1. Tekintse át a dokumentum a tárolási szakaszát.
2. Több tárfiókot használja, ha adatokat vagy a sávszélesség meghaladja a kvóták.

### <a name="sql-database"></a>SQL Database
1. Tekintse át a dokumentum az SQL-adatbázis szakaszát.
2. Átmeneti hibák kezelésének újrapróbálkozási házirendje megvalósításához.
3. Egy kibővített stratégia particionálás/horizontális használják.

### <a name="sql-server-on-virtual-machines"></a>SQL Server a Virtual Machines szolgáltatásban
1. Tekintse át az SQL Server virtuális gépek szakasz ebben a dokumentumban.
2. Kövesse az előző virtuális gépekhez.
3. SQL Server magas rendelkezésre állású szolgáltatások, például az AlwaysOn használatára.

### <a name="service-bus"></a>Service Bus
1. Tekintse át a Service Bus szakasz ebben a dokumentumban.
2. Érdemes létrehozni egy tartós ügyféloldali várólista biztonsági mentéséhez.

### <a name="hdinsight"></a>HDInsight
1. Tekintse át a dokumentum a HDInsight szakaszát.
2. További rendelkezésre állási lépésekre nincs szükség helyi hibák.

