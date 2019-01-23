---
title: 'Műszaki útmutató: Helyreállítás helyi hibák esetén az Azure-ban'
description: Annak az ismertetése, és rugalmas, magas rendelkezésre állású kialakítása, hibatűrő alkalmazások, valamint a vészhelyreállítási összpontosított helyi hibák utáni Azure-ban.
author: adamglick
ms.date: 08/18/2016
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: a567b138580999c7b7a6ae8dedb244f4e37970e7
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486046"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Technikai útmutató az Azure rugalmassága: Helyreállítás helyi hibák esetén az Azure-ban

Nincsenek rendelkezésre állásának két elsődleges fenyegetések:

- A hiba az eszközök, például a meghajtók és kiszolgálók
- Kritikus erőforrások, például a maximális betöltési körülmények között számítási kimerülése

Az Azure biztosít erőforrás-kezelést, rugalmasság, terheléselosztás és ilyen körülmények magas rendelkezésre állás engedélyezéséhez particionálás kombinációját. Ezek a funkciók némelyike végrehajtása automatikusan történik az Azure-szolgáltatásokhoz. Bizonyos esetekben azonban az alkalmazás fejlesztőjének tegye számára, hogy azok néhány további munkát.

## <a name="cloud-services"></a>Cloud Services

Az Azure Cloud Services áll egy vagy több webes vagy feldolgozói szerepkörök gyűjteménye. Egy vagy több szerepkör példányai egyidejűleg is futtathatók. A konfiguráció meghatározza, hogy a példányok számát. Szerepkörpéldányok monitorozott és felügyelt keresztül a hálóvezérlő összetevőt. A hálóvezérlő észleli, és automatikusan szoftver-és hardver válaszol.

Minden szerepkör-példány fut, a saját virtuális gépet (VM), és a hálóvezérlő egy vendégügynök keresztül kommunikál. A vendégügynök erőforrás és a csomópont mérőszámokat, beleértve a virtuális gép használati, állapot, naplók, erőforrás-használat, kivételek és hibafeltételek gyűjti. A hálóvezérlő konfigurálható időközönként a vendégügynök kérdezi le, és azt a virtuális gép újraindul, ha a vendégügynök nem válaszol. Hardverhiba esetén a társított hálóvezérlő helyezi át az összes érintett szerepkörpéldány új hardver csomópontot, és újrakonfigurálja a hálózati forgalom irányítására van.

Számára, hogy ezeket a funkciókat, fejlesztők biztosítania kell, hogy az összes szolgáltatás szerepkör elkerülése érdekében a szerepkörpéldányok állapotát tárolja. Ehelyett minden perzisztens kell elérhető tartós tárolási, például az Azure Storage vagy az Azure SQL Database. Ez lehetővé teszi a kérések kezelésére szerepköröket. Azt is jelenti, hogy szerepkör példányai is leáll bármikor a szolgáltatás átmeneti vagy állandó állapotban inkonzisztenciát létrehozása nélkül.

A követelmény a szerepköröket, a külsőleg-állapotának tárolására több hatással van. Ez azt jelenti, például, hogy egy Azure Storage-táblába az összes kapcsolódó változtatás kell módosítani egy entitáscsoportot tranzakcióban, ha lehetséges. Természetesen nem mindig lehetséges, hogy az összes módosítást egy tranzakción belül. Győződjön meg arról, hogy szerepkör sikertelen példánytelepítések nem problémákat okozhat, ha azok, amelyek a szolgáltatás állandó állapota két vagy több frissítéseit hosszú ideig futó műveletek megszakítási különös gondot kell végeznie. Ha egy másik szerepkör megpróbálja próbálkozzon újra a e művelet, azt kell ennek előrejelzését, és az eset, ahol a munkahelyi részben végrehajtva kezelését.

Vegyük példaként egy szolgáltatás, amely több üzletben particionálja az adatokat. Ha egy feldolgozói szerepkör leáll, amíg, áthelyezés, szegmensek van, valószínűleg nem fejeződött be a szegmenshez való áthelyezését. Vagy az adatáthelyezési előfordulhat, hogy ismétlődik a kezdetektől a különböző feldolgozói szerepkör, potenciálisan a árva adatokat vagy adatsérülés okozza. A problémák megelőzése érdekében hosszú ideig futó műveletek egyikét vagy mindkettőt a következőket kell lennie:

- *Idempotens*: Megismételhető hatásai nélkül. Ahhoz, hogy idempotensek, egy hosszú ideig futó művelet ugyanezt a hatást, függetlenül attól, hogy hányszor hajtja azt végre, akkor is, ha a végrehajtás során megszakad, kell rendelkeznie.
- *Növekményes újraindítható*: A legutóbbi pont folytatása sikerült. Ahhoz, hogy növekményes újraindítható, egy hosszú ideig futó művelet atomi művelet kisebb sorozatát kell állnia. Tartós tárolási, is azt kell rögzíteni a folyamat előrehaladását, úgy, hogy minden ezt követő meghívási szerzi be ahol elődjének leállt.

Végül az összes hosszú ideig futó műveletek kell hívható ismételten amíg nem sikerül. Például egy üzembe helyezési művelet előfordulhat, hogy helyezi el az Azure-üzenetsort, és ezután eltávolítja az üzenetsor feldolgozói szerepkör által csak akkor, ha ez sikeres. A szemétgyűjtés karbantartási műveletek megszakad adatok létrehozásához szükség lehet.

### <a name="elasticity"></a>Rugalmasság

Az egyes szerepkörökhöz futó példányok kezdeti száma az egyes szerepkör-konfigurációjának határozza meg. A rendszergazdák először konfigurálnia kell a várható terhelés alapján két vagy több példányát futtatni kívánt szerepkörök. De egyszerűen méretezhető szerepkörpéldányok vagy le, a használati minták módosítása. Ezt megteheti manuálisan az Azure Portalon, vagy a folyamat automatizálható a Windows PowerShell, a Service Management API vagy a külső eszközök használatával. További információkért lásd: [az automatikus skálázáshoz alkalmazás](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Particionálás

Az Azure fabric controller két típusú partíciók használ:

- Egy *frissítési tartományt* -service-szerepkörpéldányok csoportok frissítésére szolgál. Azure szolgáltatáspéldányok több tartományban történő üzembe helyezi. Helybeni frissítést a a hálóvezérlő összes példányát le egyetlen frissítési tartomány számos lehetőséget kínál, frissíti őket, és újraindítja az őket a következő frissítési tartomány áthelyezése előtt. Ez a módszer megakadályozza az egész szolgáltatás nem érhető el a frissítési folyamat alatt.
- A *tartalék tartomány* hibalehetőség adódik hardver- vagy hálózati határozza meg. Minden olyan szerepkörhöz, amelynek több példányt a hálóvezérlő biztosítja, hogy a példányok között oszlanak meg több tartalék tartomány megakadályozza, hogy elkülönített hardver-meghibásodásokkal szolgáltatás megszakítása. A tartalék tartomány szabályozza a kitettség a kiszolgáló és a hibákkal szemben.

A [Azure szolgáltatásiszint-szerződés (SLA)](https://azure.microsoft.com/support/legal/sla/) garantálja, hogy különböző tartalék és frissítési tartományokba, legalább két webes szerepkör példányai van telepítve, ha rendelkeznek külső kapcsolatokat az idő legalább 99,95 %-os. Ellentétben a frissítési tartományok nincs lehetőség a tartalék tartományok száma. Az Azure automatikusan foglalja le a tartalék tartományok és azok között osztja el a szerepkör példányai. At legalább az első két példánya minden kerüljenek a különböző hibatűrési és frissítési tartományokban tárolja, győződjön meg arról, hogy minden olyan szerepkörhöz legalább két példányt megfelelnek-e a szolgáltatói szerződés. Ez a következő ábrán jelölt.

![Tartalék tartomány elkülönítési egyszerűsített nézete](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Terheléselosztás

Az összes bejövő forgalmat a webes szerepkör áthalad egy állapot nélküli terheléselosztó, amely ügyfélkéréseket a szerepkör példányai között osztja el. Egyéni szerepkör példányai nem rendelkezik a nyilvános IP-címeket, és azok nem közvetlenül címezhetővé válnak az internetről. Webes szerepkörök nélküliek, így bármely ügyfél kérés átirányítható, bármilyen szerepkör-példány. A [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) esemény jelenik meg a 15 másodpercenként. Ezzel jelezheti, hogy a szerepkör készen áll a forgalom fogadásához, vagy legyen az elfoglalt, és a load balancer rotációból kell venni.

## <a name="virtual-machines"></a>Virtuális gépek

Az Azure Virtual Machines, a platformszolgáltatás (PaaS) szerepkörök több szempontból viszonyítva magas rendelkezésre állású számítási platform eltér. Bizonyos esetekben, tegye a magas rendelkezésre állás biztosítása érdekében további munkát.

### <a name="disk-durability"></a>Lemez tartóssága

PaaS szerepkörpéldányok, ellentétben a virtuális gép meghajtókon tárolt adatokat az állandó akkor is, ha a virtuális gép van más helyre. Az Azure virtual machines létező Virtuálisgép-lemezek használjuk az Azure Storage-blobokat. Azure Storage rendelkezésre állási jellemzőinek miatt a virtuális gépek meghajtókon tárolt adatokat is magas rendelkezésre állású.

Vegye figyelembe, hogy D meghajtó (a Windows VM-EK) az ezen szabály alól. D meghajtó a rack kiszolgálón, amelyen a virtuális gép tényleges fizikai tárhelyet, és az adatok elvesznek, ha a virtuális gép újraindul. D meghajtó csak ideiglenes tároló szól. A Linux Azure "általában" (de nem mindig) tesz elérhetővé a helyi ideiglenes lemez /dev/sdb blokk eszközként. Gyakran csatlakoztatva van az Azure Linux-ügynök által /mnt/resource vagy /mnt csatlakoztatási pontokra (/etc/waagent.conf keresztül konfigurálható).

<!-- markdownlint-disable MD024 -->

### <a name="partitioning"></a>Particionálás

Az Azure natív módon megért a rétegek egy PaaS-alkalmazásban (webes és feldolgozói szerepkörök), és így tudja megfelelően kioszthatja tartalék és frissítési tartományok között. Ezzel szemben a rétegek egy infrastruktúra-szolgáltatás (IaaS) alkalmazásként manuálisan definiálni kell keresztül a rendelkezésre állási csoportok. A rendelkezésre állási csoportok szükségesek alatt IaaS SLA-t.

![Rendelkezésre állási csoportok az Azure-beli virtuális gépek](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

A fenti ábrán az Internet Information Services (IIS) szint (amely a web app réteget módon működik) és az SQL-szint (amely egy adatrétegbeli módon működik) hozzá van rendelve másik rendelkezésre állási csoportokat. Ez biztosítja, hogy rendelkezik-e az egyes szintek összes példányát hardverredundancia osztja meg a virtuális gépek a tartalék tartományok között, és, hogy teljes szinten nem egy frissítés alatt álló alkotásait.

### <a name="load-balancing"></a>Terheléselosztás

A virtuális gépek elosztott azok között forgalmat kell rendelkeznie, ha egy alkalmazás és a terhelés elosztása a virtuális gépek között egy adott TCP vagy UDP-végpontnak kell csoportosítja. További információkért lásd: [terheléselosztási virtuális gépek](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Ha a virtuális gépek bemeneti más forrásból (például egy üzenetsor-kezelési mechanizmust) kap, a load balancer, nem szükséges. A load balancer alapszintű állapot-ellenőrzése alapján határozza meg, hogy kell-e forgalmat küldeni a csomópontra. Az is lehet létrehozni a saját mintavételek megvalósításához és alkalmazásspecifikus mérőszámok, amelyek meghatározzák, hogy a virtuális gép jár-e a forgalmat.

## <a name="storage"></a>Tárhely

Az Azure Storage szolgáltatása a referenciakonfiguráció hosszú élettartamú adatok az Azure-hoz. Blob, table, queue és virtuális gép lemezes tárolás biztosít. Replikáció és az erőforrás-kezelés együttes használatával magas rendelkezésre állás egyetlen adatközponton belül. Az Azure Storage rendelkezésre állási SLA biztosítja azt, hogy legalább 99,9 %-ában:

- Helyesen formázott kérelmeket szeretne hozzáadni, update, olvassa el és az adatok törlésével fogja sikeres és helyesen feldolgozni.
- Storage-fiókok elérhetőséget, az internetes átjárónkhoz.

### <a name="replication"></a>Replikáció

Az Azure Storage segíti az adatok tartós megőrzését között teljesen független fizikai tárhelyek alrendszerei a régión belül több példányának különböző meghajtókon található összes adat megőrzése. Szinkron módon rendszer replikálja az adatokat, és mindegyik példányt véglegesítése előtt a rendszer arra vonatkozik, az írási. Az Azure Storage szolgáltatás erősen konzisztens, ami azt jelenti, hogy olvasási garantáltan a legújabb írást tükrözik. Adatok másolatainak emellett folyamatosan vizsgált észlelése és javítása bit rothadás, egy gyakran tényezőkből fenyegetés tárolt adatok sértetlenségét.

Szolgáltatások csak az Azure Storage használatával előnyeit a replikációból. A szolgáltatás fejlesztői nem kell helyi hiba helyreállítása további munkát.

### <a name="resource-management"></a>Erőforrás-kezelés

2014. május, után létrehozott Storage-fiókok legfeljebb 500 TB-ig (a korábbi maximális volt 200 TB) növelhető. Ha további tárterületre szükség, alkalmazásokat kell kialakítani, hogy több tárfiók használata.

### <a name="virtual-machine-disks"></a>Virtuálisgép-lemezek

Az Azure Storage adná ugyanazokat tartósság és a méretezhetőség tulajdonság a Blob storage, lapblob egy virtuálisgép-lemez van tárolva. Ez a kialakítás teszi az adatok egy virtuálisgép-lemez állandó, még akkor is, ha a virtuális Gépet futtató kiszolgáló meghibásodik, és újra kell indítani a virtuális gép egy másik kiszolgálón.

## <a name="database"></a>Adatbázis

### <a name="sql-database"></a>SQL-adatbázis

Az Azure SQL Database-adatbázist kínál szolgáltatásként. Lehetővé teszi alkalmazások gyors üzembe helyezése, helyezze be az adatokat, és a relációs adatbázisok lekérdezése. Az ismerős SQL-kiszolgálói szolgáltatásairól és funkcióiról, számos hardver, konfigurációs, javítási és rugalmasság terhe paltformfüggetlen közben biztosít.

> [!NOTE]
> Az Azure SQL Database nem biztosít az SQL Server-az-egyhez funkcióparitás. Az követelmények –, amely rendelkezik egyedi alkalmas a felhőalapú alkalmazások (rugalmasan méretezhető, adatbázis-szolgáltatás karbantartási költségek csökkentéséhez és így tovább) külön készletét teljesítéséhez rendelkezik szolgál. További információkért lásd: [felhőalapú SQL Server option: Az Azure SQL (PaaS) adatbázis vagy SQL Server Azure virtuális gépeken (IaaS)](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).

#### <a name="replication"></a>Replikáció

Az Azure SQL Database csomópont-szintű hiba beépített rugalmasságot biztosít. Az összes írási művelet az adatbázisba hátterének legalább két csomóponttal keresztül egy kvórum véglegesítési technika automatikusan replikálva van. (Az elsődleges és másodlagos legalább egy meg kell erősítenie, hogy a tevékenység írt a tranzakciónapló, mielőtt a tranzakció sikeres minősül, és adja vissza.) Csomóponthiba esetén az adatbázis automatikusan átadja a feladatokat a másodlagos replikára. Ez azt eredményezi, hogy egy átmeneti kapcsolat megszakadása, az ügyfélalkalmazások számára. Ebből kifolyólag minden Azure SQL Database-ügyfelek musí implementovat valamilyen átmeneti kapcsolati kezelését. További információkért lásd: [az újrapróbálkozási mechanizmushoz adott](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Erőforrás-kezelés

Minden adatbázis létrehozása után úgy van konfigurálva, és a egy méretének felső korlátja. A jelenleg rendelkezésre álló maximális mérete 1 TB-os (korlátok eltérő méretű lásd a szolgáltatási réteg alapján [szolgáltatás és az Azure SQL Database-adatbázisok teljesítményszintek](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Egy adatbázis találatok száma a felső méretkorlátot, elutasítja a további INSERT nebo UPDATE parancsok. (Lekérdezés, és az adatok törlése az továbbra is lehetséges.)

Egy adatbázison belül az Azure SQL Database használ a háló-erőforrások kezeléséhez. Azonban a hálóvezérlő helyett használja egy körgyűrűs topológia a hibák észlelése érdekében. Minden replika egy fürtben két szomszédok rendelkezik, és azt észlelte, amikor azok felelős. Ha egy replika leáll, szomszédjainak aktiválása egy újrakonfigurálási ügynök újra létre egy másik gépen. Annak biztosítása érdekében, hogy egy logikai kiszolgálót nem túl sok erőforrást használja a gépén vagy haladhatja meg a gép fizikai korlátok motor szabályozás biztosítja.

### <a name="elasticity"></a>Rugalmasság

Ha az alkalmazás több, mint az 1 TB-os adatbázis korlátozása, azt meg kell valósítania egy kibővített megközelítés. Horizontális felskálázása az Azure SQL Database particionálásával manuálisan, más néven horizontális skálázás, adatokat több SQL-adatbázisban. A horizontális felskálázás megközelítés méretek mellett közel lineáris költség növekedés elérése lehetőséget biztosít. Rugalmas növekedést vagy igény szerinti kapacitás növekményes költségekkel igény szerint növelhető, mert az adatbázis átlagos tényleges mérete alapján használja egy nap, nem a maximális mérete alapján számolunk fel.

## <a name="sql-server-on-virtual-machines"></a>SQL Server on Virtual Machines

Telepíti az SQL Server (2014 vagy újabb verzió) az Azure Virtual machines szolgáltatásban, használhatja az SQL Server hagyományos rendelkezésre állási funkcióit is igénybe vehet. Ezek a funkciók közé tartozik az AlwaysOn rendelkezésre állási csoportok és az adatbázis-tükrözés. Vegye figyelembe, hogy az Azure virtuális gépek, tárolási és hálózatkezelési egy helyszíni informatikai infrastruktúrával nem virtualizált eltérő működési jellemzői. Egy magas rendelkezésre állás/vészhelyreállítás az Azure-ban (HA/DR) az SQL Server-megoldás sikeres megvalósítását megköveteli, hogy ezek a különbségek megértéséhez, és a megoldás kialakításakor megtervezése.

### <a name="high-availability-nodes-in-an-availability-set"></a>Magas rendelkezésre állású csomópontok egy rendelkezésre állási csoportban

Egy magas rendelkezésre állású megoldások megvalósításának az Azure-ban, a rendelkezésre állási csoportot az Azure segítségével a magas rendelkezésre állású csomópontok helyezze külön tartalék tartományokban és frissítési tartományokban tárolja. A rendelkezésre állási csoport tisztázzuk egy Azure fogalom. Fontos, hogy ellenőrizze, hogy az adatbázisok valóban magas rendelkezésre állású, legyen szó az AlwaysOn rendelkezésre állási csoportok, adatbázis-tükrözés vagy valami mást a követendő ajánlott. Ha nem követi az ajánlott eljárás, valószínűleg hamis feltételezve, hogy a rendszer magas rendelkezésre állású. De a valóságban a végzett, a csomópontok összes sikertelen lehet egyidejűleg mert mobilak ugyanabban a tartalék tartományban található, az Azure-régióban kell elhelyezni.

Ez a javaslat nem is alkalmazható a naplóküldésben. Egy vész-helyreállítási funkció, győződjön meg, hogy a kiszolgáló fut-e külön Azure-régióban. Definíció szerint ezekben a régiókban a külön tartalék tartományokban.

Az Azure Cloud Services üzembe helyezett virtuális gépek a klasszikus portálon keresztül kell ugyanabban a rendelkezésre állási csoportban telepítenie kell őket az ugyanazon felhőszolgáltatásban. Az Azure Resource Manager (aktuális portálon) üzembe helyezett virtuális gépek nem kell ezt a korlátozást. A klasszikus portálon üzembe helyezett virtuális gépek az Azure Cloud Service, ugyanazon a Felhőszolgáltatáson csomópontok csak az azonos rendelkezésre állási csoport vehet részt. Emellett a Cloud Services virtuális gépeket annak érdekében, hogy az IP-címek megőriznek felültelepítés után is ugyanabban a virtuális hálózatban kell lennie. Ezzel elkerülhető a DNS frissítési megszakadását.

### <a name="azure-only-high-availability-solutions"></a>Azure-only: Magas rendelkezésre állású megoldások

AlwaysOn rendelkezésre állási csoportok vagy az adatbázis-tükrözés rendelkezhet egy magas rendelkezésre állású megoldás az SQL Server-adatbázisait az Azure-ban.

A következő ábra azt mutatja be, az architektúra az Azure virtuális gépeken futó AlwaysOn rendelkezésre állási csoportok. Ez az ábra a témával kapcsolatos, a részletes cikkben hibaállapota [az Azure virtuális gépeken futó SQL Server magas rendelkezésre állás és vészhelyreállítás helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![AlwaysOn rendelkezésre állási csoportok a Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Az Azure Portalon az AlwaysOn-sablon használatával automatikusan is telepíthet egy AlwaysOn rendelkezésre állási csoportok üzembe helyezés – teljes körű Azure virtuális gépeken. További információkért lásd: [SQL Server AlwaysOn ajánlatot a Microsoft Azure Portal Gallery](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/).

Az alábbi diagram bemutatja, hogy az adatbázis-tükrözés az Azure Virtual machines szolgáltatásban. A részletes témakörből is hibaállapota [az Azure virtuális gépeken futó SQL Server magas rendelkezésre állás és vészhelyreállítás helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Adatbázis-tükrözés a Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> Mindkét architektúrát tartományvezérlőt igényelnek. Azonban az adatbázis-tükrözés, lehetősége tartományvezérlő szükségtelenné teszi a kiszolgálói tanúsítványok használatával.

## <a name="other-azure-platform-services"></a>Más Azure platformszolgáltatások

Az Azure-ban készített alkalmazások számára előnyös platform képességei a helyi hibák utáni helyreállításhoz. Bizonyos esetekben az adott forgatókönyv a rendelkezésre állás növelése érdekében bizonyos műveleteket hajthatja végre.

### <a name="service-bus"></a>Service Bus

Az Azure Service Bus egy átmeneti szolgáltatáskimaradás veszélye, fontolja meg egy ügyféloldali tartós üzenetsor létrehozása. Ez ideiglenesen használja egy másik, a helyi tárolási mechanizmus, amely nem adható hozzá a Service Bus-üzenetsorba üzenetek tárolásához. Az alkalmazás eldöntheti, hogyan legyen kezelve az ideiglenesen tárolt üzenetek, a szolgáltatás visszaállítása után. További információkért lásd: [ajánlott eljárások a teljesítmény használatával a Service Bus közvetítőalapú üzenettovábbítás](/azure/service-bus-messaging/service-bus-performance-improvements/) és [a Service Bus (katasztrófa utáni helyreállítás)](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight

Az Azure Blob storage-ban alapértelmezés szerint az Azure HDInsight társított adatokat tárolja. Az Azure Storage a Blob storage magas rendelkezésre állást és tartósságot tulajdonságát meghatározza. A több csomópontos feldolgozás Hadoop MapReduce-feladatok társított akkor fordul elő, az egy átmeneti Hadoop elosztott fájlrendszer (HDFS), amely ki van építve, amikor HDInsight kell azt. Egy MapReduce-feladatot az eredmények is tárolása az Azure Blob storage, alapértelmezés szerint, hogy a feldolgozott adatok tartósságát és magas rendelkezésre állású marad, a Hadoop-fürt az eltávolítása után. További információkért lásd: [HDInsight (katasztrófa utáni helyreállítás)](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Helyi hibák Ellenőrzőlisták

### <a name="cloud-services-checklist"></a>Cloud Services ellenőrzőlista

1. Tekintse át a Cloud Services szakasz ebben a dokumentumban.
2. Állítsa be legalább két példányt az egyes szerepkörökhöz.
3. Továbbra is fennáll a tartós tárolási, nem a szerepkörpéldányok állapotát.
4. Helyesen kezeli a StatusCheck eseményét.
5. Összefüggő változások zabalit do tranzakciók, amikor csak lehetséges.
6. Győződjön meg arról, hogy a feldolgozói szerepkör feladatok idempotensek és újraindítható.
7. Továbbra is, hogy műveleteket hívjon meg, amíg nem sikerül.
8. Fontolja meg az automatikus skálázási stratégiákra.

### <a name="virtual-machines-checklist"></a>Virtuális gépek ellenőrzőlista

1. Tekintse át a virtuális gépekre vonatkozó szakaszban ebben a dokumentumban.
2. Ne használja a D meghajtó hosszú távú adattárolásra.
3. A gépeket egy rendelkezésre állási csoportba egy szolgáltatási rétegben található.
4. Terheléselosztás és a választható mintavétel konfigurálása.

### <a name="storage-checklist"></a>Tárolási ellenőrzőlista

1. Tekintse át a Storage szakasz ebben a dokumentumban.
2. Több tárfiók használata, ha adatokat vagy a sávszélesség túllépi kvótákat.

### <a name="sql-database-checklist"></a>Az SQL Database ellenőrzőlista

1. Tekintse át az SQL Database szakasz ebben a dokumentumban.
2. Átmeneti hibák kezeléséhez újrapróbálkozási szabályzat megvalósítása.
3. Particionálási és horizontális particionálás egy horizontális felskálázási stratégiát használja.

### <a name="sql-server-on-virtual-machines-checklist"></a>Az SQL Server virtuális gépek ellenőrzőlista használata

1. Tekintse át az SQL Server, a virtuális gépekre vonatkozó szakaszban ebben a dokumentumban.
2. Kövesse a fenti javaslatok a virtuális gépekhez.
3. Az SQL Server magas rendelkezésre állású szolgáltatások, például AlwaysOn használata.

### <a name="service-bus-checklist"></a>A Service Bus ellenőrzőlista

1. Tekintse át a Service Bus szakasz ebben a dokumentumban.
2. Érdemes létrehozni egy ügyféloldali üzenetsorban biztonsági mentéséhez.

### <a name="hdinsight-checklist"></a>HDInsight-feladatlista

1. Tekintse át a HDInsight szakasz ebben a dokumentumban.
2. További rendelkezésre állási lépés nem szükségesek helyi hibák esetén.

<!-- markdownlint-enable MD024 -->
