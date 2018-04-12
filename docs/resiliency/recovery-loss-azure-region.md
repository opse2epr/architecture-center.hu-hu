---
title: Egy Azure-régiót származó helyreállítása
description: 'A következő cikket: ismertetése és rugalmas, magas rendelkezésre állású, hiba hibatűrő alkalmazásokhoz tervezéséhez, valamint vészhelyreállítás tervezése'
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: f551e8af8aece8aa30abfba2438c41c3944209bd
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Az Azure rugalmasságát műszaki útmutató: a régió kiterjedő szolgáltatás szüneteltetése helyreállítás
Azure oszlik fizikai és logikai egységek régiók hívása. A régió közel egy vagy több adatközpontok áll. 

Ritka esetekben is lehet, hogy egy teljes régióban létesítményekben nem érhető el, például hálózati hibák miatt válhat. Vagy létesítményekben elvesznek, teljes mértékben, például természeti katasztrófa miatt. Ez a szakasz ismerteti az Azure képességeit régiók elosztott alkalmazások létrehozásához. Ilyen terjesztési segít a lehetősége, hogy az egy régióban hiba hatással lehet más régiókban minimalizálása érdekében.

## <a name="cloud-services"></a>Felhőszolgáltatások
### <a name="resource-management"></a>Erőforrás-kezelés
Minden cél régióban külön felhőalapú szolgáltatás létrehozása, és majd közzététele a központi telepítési csomag minden felhőalapú szolgáltatás által terjesztheti számítási példányokért régiók között. Vegye figyelembe azonban, hogy osztja a forgalom között felhőszolgáltatások különböző régiókban kell megvalósítani az alkalmazás fejlesztőjének vagy egy forgalom felügyeleti szolgáltatáshoz.

Tartalék szerepkörpéldányokat előre telepítendő vész-helyreállítási számának meghatározása a kapacitástervezés fontos eleme. A teljes körű másodlagos központi telepítés rendelkezik biztosítja, hogy kapacitás már érhető el, ha szükséges, azonban ez gyakorlatilag megduplázódik költsége. A megszokott mintát követi, hogy egy kis, másodlagos telepítés, elég nagy kritikus szolgáltatások futtatásához. Ez kis másodlagos telepítési érdemes, mind a kapacitás, és a másodlagos környezet konfigurációjától tesztelési.

> [!NOTE]
> Az előfizetési kvóta nincs kapacitás garanciát. A kvóta egyszerűen csak egy értéket. Kapacitás biztosításához, a szükséges szerepkörök száma definiálni kell a szolgáltatásmodellben, és a szerepköröket kell telepíteni.
> 
> 

### <a name="load-balancing"></a>Terheléselosztás
Régiók közötti forgalom terheléselosztásához kell rendelkeznie a forgalom-kezelési megoldás. Azure biztosít [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). A kihasználásához, külső szolgáltatások hasonló forgalom felügyeleti képességeket biztosítják.

### <a name="strategies"></a>Stratégiák
Alternatív stratégiák számos különböző régiókban elosztott számítási végrehajtásához érhetők el. Ezek a meghatározott üzleti követelményeknek és a körülmények között az alkalmazás kell igazítani. Magas szinten a módszer a következő kategóriákba sorolhatók osztható:

* **Telepítse újra a katasztrófa**: ennek a megközelítésnek az az alkalmazás van újratelepítése teljesen új katasztrófa időpontjában. Ez az a nem kritikus fontosságú alkalmazások, amelyek nem igényelnek garantált helyreállításkor megfelelő.
* **Tartalék (aktív/passzív) mozog**: egy másodlagos üzemeltetett szolgáltatás létrejön egy másik régióban, és minimális kapacitás biztosításához telepített szerepkörök; azonban a szerepkörök nem kap a termelési forgalmat. Ez a módszer akkor hasznos, nem forgalom szét régiók készített alkalmazások.
* **Tartalék (aktív) közbeni**: célja, hogy az alkalmazás üzemi terhelés több régióba fogadására. A felhőszolgáltatások minden régióban vész-helyreállítási célokra szükségesnél nagyobb kapacitást is megadhatók. Azt is megteheti, a felhőalapú szolgáltatások méretezése előfordulhat, hogy szükség esetén egy katasztrófa és a feladatátvétel időpontjában ki. Ezt a módszert használja az alkalmazás tervét jelentős befektetési igényel, de jelentős előnyt rendelkezik. Ezek közé tartozik a kis- és garantált helyreállítási idő, a folyamatos tesztelés az összes helyreállítási helyeken, és a kapacitás hatékony kihasználását.

Elosztott kialakítás teljes leírása jelen dokumentum nem terjed van. További információkért lásd: [vész-helyreállítási és magas rendelkezésre állás, az Azure-alkalmazások](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Virtuális gépek
Egy szolgáltatási (IaaS) virtuális gépként (VM) infrastruktúra helyreállítási hasonlít a platform, a szolgáltatás (PaaS) számítási sok tekintetben helyreállítási. Fontos különbségek vannak, azonban annak köszönhető, hogy az infrastruktúra-szolgáltatási virtuális gép a virtuális gép lemez és a virtuális Gépet tartalmaz.

* **Több régióban biztonsági mentések, amelyek a konzisztens alkalmazás létrehozása az Azure Backup használatával**.
  [Azure biztonsági mentés](https://azure.microsoft.com/services/backup/) lehetővé teszi az ügyfelek alkalmazás konzisztens biztonsági másolatok létrehozása a több virtuális gép lemezre, és régiók közötti replikáció a biztonsági mentések támogatásához. Ehhez kiválasztásával földrajzi-replikálja a mentési tároló létrehozásának időpontjában. Vegye figyelembe, hogy a replikáció a mentési tároló létrehozása idején kell konfigurálni. Később beállítható. A régió nem vesztek el, ha Microsoft lesz a biztonsági mentések elérhetővé teszi az ügyfél számára. Ügyfelek állíthatja helyre a konfigurált visszaállítási pontok bármelyikét is.
* **Az operációsrendszer-lemeze a adatlemez külön**. Az infrastruktúra-szolgáltatási virtuális gépek fontos szempont, hogy, hogy az operációsrendszer-lemez nem módosítható a virtuális gép újbóli létrehozása nélkül. Ez nem hiba esetén pedig a helyreállítási stratégia katasztrófa utáni újratelepíteni. Előfordulhat azonban, a probléma, ha a meleg tartalék módszer segítségével tartalékkapacitás. Ennek megfelelően alkalmazza, rendelkeznie kell a megfelelő operációsrendszer-lemez, az elsődleges és másodlagos helyekre telepített, és az alkalmazásadatok egy különálló meghajtón kell tárolni. Lehetőség szerint használja a szabványos operációs rendszer-konfigurációkat, amelyek a is megadható. Egy feladatátvétel után kell majd csatol az adatmeghajtó a meglévő infrastruktúra-szolgáltatási virtuális gépeket a másodlagos tartományvezérlő. AzCopy segítségével az adatok lemez(ek) pillanatkép-készítési egy távoli helyre másolja.
* **Vegye figyelembe a lehetséges konzisztenciabeli problémákat több virtuális gép lemezeivel földrajzi feladatátvételt követően**. Virtuális gépek lemezei, Azure Storage blobs van megvalósítva, és az azonos georeplikáció jellemző rendelkezik. Ha [Azure Backup](https://azure.microsoft.com/services/backup/) van használva nincsenek eredő garanciát nem helyét a lemezeken, mert georeplikáció aszinkron, és egymástól függetlenül replikálja. Az egyes virtuális gépek lemezei garantáltan az összeomlás-konzisztens állapot földrajzi-feladatátvétel után, de nem konzisztens helyét a lemezeken. Ez problémákat okozhat egyes esetekben (például esetén a lemez csíkozást).

## <a name="storage"></a>Tárolás
### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Helyreállítási blob, table, várólista és VM lemezegységet Georedundáns tárolás használatával
Az Azure, blobok, táblák, üzenetsorok és virtuális gép lemezek, az összes georeplikált alapértelmezés szerint. Ez a neve, Georedundáns tárolás (GRS). Georedundáns tárolás adatait miles egymástól belül egy adott földrajzi régió egy párosított adatközpont több száz replikálja. Adja meg a további tartósságot, ha van a fő adatközpontok katasztrófa Georedundáns célja. Microsoft vezérlők feladatátvétel esetén, és a feladatátvételi korlátozódik jelentős katasztrófa, amelyben az eredeti elsődleges helyre akkor számít elértnek helyreállíthatatlan elfogadható időn belül. Bizonyos esetekben több napig is lehet. Bár a szinkronizálás időköze nem még mutatja be egy szolgáltatásiszint-szerződés adatok általában replikált néhány percen belül.

Földrajzi-feladatátvétel esetén lesz hogyan érhető el a fiók nem változik (az URL-cím és a fiók kulcs esetén nem változik). A tárfiók, azonban szerepelni fog egy másik régióban feladatátvételt követően. Ez hatással lehet a storage-fiók regionális kapcsolatot igénylő alkalmazások. Még a szolgáltatásokat és alkalmazásokat, amelyek nem igényelnek a tárfiók ugyanabban az adatközpontban a kereszt-datacenter késleltetés és a sávszélesség-költségek előfordulhat, hogy lehet kényszerítő forgalom ideiglenesen áthelyezését a feladatátvételi régió. Ez egy általános vész-helyreállítási stratégiát az sikerült számításba.

Georedundáns által biztosított automatikus feladatátvétel mellett Azure vezetett be olyan szolgáltatás, amely olvasási hozzáférést biztosít a másodlagos tárolási helye az adatok másolatát. Ennek meghívása az írásvédett Georedundáns tárolás (RA-GRS).

Georedundáns, mind az RA-GRS tárolóira vonatkozó további információkért lásd: [Azure Storage replikációs](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>A Georeplikáció régió leképezéseket:
Fontos tudni, hogy hol található az adatok georeplikált, ahhoz, hogy tudja, honnan központi telepítése az adatok tárhelyét regionális kapcsolat igénylő más példányát. További információ: [Azure-régiókat párosítva](/azure/best-practices-availability-paired-regions).

### <a name="geo-replication-pricing"></a>A Georeplikáció árképzési:
Az Azure Storage aktuális árképzési georeplikáció tartalmazza. Ez a lehetőség Georedundáns tárolás (GRS). Ha nem szeretné, hogy az adatok georeplikált georeplikáció bármikor letilthatja a fiókjához. Ezt nevezik a helyileg redundáns tárolás és Georedundáns képest kedvezményes áron fel van töltve.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Ha egy földrajzi feladatátvétel történt meghatározása
Egy földrajzi feladatátvétel esetén ez lesznek közzétéve a [Azure az állapotjelző irányítópulthoz](https://azure.microsoft.com/status/). Alkalmazások is létrehozható egy automatizált azt, ez azonban a tárfiók földrajzi régió figyelésével. Ez más helyreállítási műveletek, például a számítási erőforrásokat, ahol azok tárolási áthelyezése a földrajzi régióban aktiválási kiváltásához is használható. Is elvégezheti a lekérdezés ezen a szolgáltatáskezelő API, használatával [lekérni a Tárfiók tulajdonságai](https://msdn.microsoft.com/library/ee460802.aspx). A kapcsolódó tulajdonságok a következők:

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>Virtuális gépek lemezei és a földrajzi-feladatátvétel
A szakaszban bemutatott VM lemezeken, nincsenek nem garantálja az adatok konzisztenciájának virtuális gép lemezeinek egy feladatátvétel után. Ahhoz, hogy a biztonsági mentések helyességét, például a Data Protection Manager biztonsági mentési termék használatával biztonsági mentése és visszaállítása alkalmazásadatokat.

## <a name="database"></a>Adatbázis
### <a name="sql-database"></a>SQL Database
Az Azure SQL Database is tartalmaz kétféle helyreállítási: Georedundáns helyreállítás és aktív Georeplikáció.

#### <a name="geo-restore"></a>Georedundáns helyreállítás
[Georedundáns helyreállítás](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) Basic, Standard és Premium adatbázisok érhetők el. Az alapértelmezett helyreállítási lehetőséget biztosít, ha az adatbázis nem érhető el a régióban, ahol az adatbázis tárolása esemény miatt. Hasonló pont időponthoz kötött visszaállítás, Georedundáns helyreállítás támaszkodik adatbázis az georedundáns Azure-tárfiókba. Visszaállítja a georeplikált biztonsági másolat, és ezért is lehetséges legyen a tárolási kimaradások esetén az elsődleges régióban. További részletekért lásd: [egy Azure SQL Database vagy feladatátvételi visszaállításához a másodlagos](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Aktív georeplikáció
[Aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/) érhető el az összes adatbázis-rétegek. Az alkalmazásokat, amelyek szigorúbb helyreállítási követelmények Georedundáns helyreállítás kínálhat, mint a tervezték. Aktív Georeplikációt használ, létrehozhat négy olvasható másodlagos adatbázis a különböző régiókban található kiszolgálók. Feladatátvételt a másodlagos adatbázisok bármelyikét is kezdeményezhető. Aktív Georeplikáció ezenkívül az alkalmazás frissítését, vagy áthelyezheti forgatókönyvek támogatása, valamint a terheléselosztás csak olvasható munkaterhelések esetén használható. További információkért lásd: [Georeplikáció konfigurálása](/azure/sql-database/sql-database-geo-replication-portal/) és [feladatátvételt a másodlagos adatbázis](/azure/sql-database/sql-database-geo-replication-failover-portal/). Tekintse meg [felhő vész-helyreállítási aktív Georeplikáció használatával az SQL-adatbázis egy alkalmazás](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) és [SQL adatbázis aktív Georeplikációhasználófelhőalapúalkalmazásokműködésközbenifrissítésekkezelése](/azure/sql-database/sql-database-manage-application-rolling-upgrade/)vonatkozó részletek tervezése és megvalósítása alkalmazásokat és a frissítés állásidő nélkül.

### <a name="sql-server-on-virtual-machines"></a>SQL Server a Virtual Machines szolgáltatásban
Számos lehetőséget a helyreállításhoz, majd az SQL Server 2012 (és újabb verziók) futtató Azure virtuális gépek magas rendelkezésre állású érhetők el. További információkért lásd: [az SQL Server Azure virtuális gépek magas rendelkezésre állási és vészhelyreállítási helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Egyéb Azure platform szolgáltatásaiból
Ha megpróbálja futtatni a felhőalapú szolgáltatás több Azure-régiók, figyelembe kell vennie a megvalósítását az egyes a függőségek. Az alábbi szakaszok a szolgáltatással kapcsolatos útmutató feltételezi, hogy az egy másik Azure-adatközpontban kell használnia az azonos Azure-szolgáltatás. Ez magában foglalja a konfigurációs és a replikációs adatok feladatok.

> [!NOTE]
> Bizonyos esetekben lépések segítségével egész adatközpont esemény helyett a szolgáltatásspecifikus szolgáltatáskimaradás elhárítása érdekében. Az alkalmazás szempontjából, egy szolgáltatás-specifikus leállás lehet, mint korlátozása csak, és a szolgáltatás ideiglenesen áttelepítése egy másodlagos Azure régióra igényel.
> 
> 

### <a name="service-bus"></a>Service Bus
Az Azure Service Bus által használt Azure-régiók nem fedik egyedi névtér. Az első követelmény úgy beállítani a szükséges szolgáltatásbusz-névterek az másodlagos régióban. Van azonban is a várólistára helyezett üzenetek a tartóssági szempontjai. Nincsenek több stratégiák üzenetek replikálása Azure-régiók. A replikálási gyakorlata és más vész-helyreállítási stratégiát a részletekért lásd: [ajánlott eljárások az alkalmazások a Service Bus kimaradások és vészhelyzetek szigetelő](/azure/service-bus-messaging/service-bus-outages-disasters/). Más rendelkezésre állási lehetőségekért lásd: [Service Bus (rendelkezésre állási)](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>App Service
Az Azure App Service alkalmazás, például webes alkalmazásokat, illetve a Mobile Apps át egy másodlagos Azure-régió, közzététel érhető el a webhely biztonsági másolatot kell rendelkeznie. Ha a szolgáltatáskimaradás nem tartalmaz, amely a teljes Azure-adatközpontban, esetleg FTP segítségével töltse le a webhely tartalmát egy nemrég készült biztonsági másolatából. Ezután hozzon létre egy új alkalmazást a másodlagos régióban, kivéve, ha korábban ezt a tartalékkapacitás. A hely közzététele az új területre, és a szükséges konfigurációs módosításokat. Ezek a változások tartalmazhatnak, adatbázis-kapcsolati karakterláncok vagy más régióban-specifikus beállításokat. Ha szükséges, a webhely SSL-tanúsítvány hozzáadása, és módosítsa a DNS CNAME rekord úgy, hogy az egyéni tartománynév mutat, az újratelepített Azure webes alkalmazás URL-CÍMÉT.

### <a name="hdinsight"></a>HDInsight
A hdinsight eszközzel kapcsolatos adatok az Azure Blob Storage alapértelmezés szerint tárolja. HDInsight megköveteli, hogy a Hadoop-fürtök MapReduce-feladatok feldolgozásában, közös elhelyezése szükséges a tárfiókot, amely tartalmazza az adatokat az elemezni ugyanabban a régióban. A georeplikáció elérhető Azure Storage szolgáltatást használja, feltéve érheti el az adatokat a másodlagos régióban, ahol az adatok replikálása Ha valamilyen okból az elsődleges régióban már nem érhető el. A régió, ahol az adatok replikálja, és folytatni azt is létrehozhat egy új Hadoop-fürt. Más rendelkezésre állási lehetőségekért lásd: [HDInsight (rendelkezésre állási)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>SQL Reporting (SQL-jelentés)
Most végezze el a kívánt Azure-régiót elvesztését különböző Azure-régiók több SQL-példány van szükség. Ezek SQL-példány kell ugyanazon adatokat érik el, és adatokat kell rendelkeznie a saját helyreállítási terv legyen katasztrófahelyzet esetén. Minden jelentés RDL-fájl külső biztonsági másolatai is kezelheti.

### <a name="media-services"></a>Media Services
Az Azure Media Services van egy másik helyreállítási megközelítés kódolás és adatfolyamként történő továbbítását. Adatfolyam-általában több kritikus regionális kimaradás során. Ez az előkészítéséhez kell két különböző Azure-régiók Media Services-fiók. A kódolt tartalom mindkét régió kell elhelyezni. Adott meghibásodás során a másodlagos régióba irányíthatja át a folyamatos átviteli forgalmat. Kódolás bármely Azure régióban hajtható végre. Ha kódolás időérzékeny, például az élő esemény feldolgozása során, egy másodlagos adatközpontba feladatok küldéséhez esetén készen kell állnia.

### <a name="virtual-network"></a>Virtuális hálózat
Konfigurációs fájlokat adja meg a leggyorsabban egy másodlagos Azure régióra virtuális hálózat létrehozása. Az elsődleges Azure-régió, a virtuális hálózathoz való beállítása után [exportálhatja a virtuális hálózati beállításait](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) az adott hálózathoz hálózati konfigurációs fájlt. Az elsődleges régióban kimaradás esetén [visszaállítani a virtuális hálózati](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) a tárolt konfigurációs fájlból. Ezután konfigurálja a más felhőszolgáltatások, virtuális gépek vagy létesítmények közötti beállítások dolgozni az új virtuális hálózat.

## <a name="checklists-for-disaster-recovery"></a>Vész-helyreállítási Ellenőrzőlisták
## <a name="cloud-services-checklist"></a>Cloud Services ellenőrzőlista
1. Tekintse át a dokumentum a Felhőszolgáltatások szakaszát.
2. Hozzon létre egy kereszt-régió vész-helyreállítási stratégiát.
3. Ismerje meg a lefoglal kapacitást a másodlagos régióban kompromisszumot.
4. Használja a forgalom útválasztási eszközök, például az Azure Traffic Manager.

## <a name="virtual-machines-checklist"></a>Virtuális gépek ellenőrzőlista
1. Tekintse át a virtuális gépek szakasz ebben a dokumentumban.
2. Használjon [Azure biztonsági mentés](https://azure.microsoft.com/services/backup/) alkalmazás konzisztens biztonsági másolatok különböző régiókban létrehozásához.

## <a name="storage-checklist"></a>Tárolási ellenőrzőlista
1. Tekintse át a dokumentum a tárolási szakaszát.
2. Ne tiltsa le a tároló-erőforrások a georeplikáció.
3. Ismerje meg a georeplikációért másodlagos régióban feladatátvételkor.
4. Hozzon létre egyéni biztonsági stratégiák a felhasználó által szabályozott feladatátvételi stratégiát.

## <a name="sql-database-checklist"></a>SQL-adatbázis ellenőrzőlista
1. Tekintse át a dokumentum az SQL-adatbázis szakaszát.
2. Használjon [Georedundáns helyreállítás](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) vagy [Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/) szükség szerint.

## <a name="sql-server-on-virtual-machines-checklist"></a>SQL Server, a virtuális gépek ellenőrzőlista
1. Tekintse át az SQL Server virtuális gépek szakasz ebben a dokumentumban.
2. A kereszt-régió AlwaysOn rendelkezésre állási csoportok vagy az adatbázis-tükrözés.
3. Felváltva biztonsági másolat, és állítsa vissza a blob-tároló.

## <a name="service-bus-checklist"></a>A Service Bus ellenőrzőlista
1. Tekintse át a Service Bus szakasz ebben a dokumentumban.
2. Konfigurálja a Service Bus-névtér egy másik régióban.
3. Fontolja meg az üzenetek egyéni replikálási gyakorlata régiók között.

## <a name="app-service-checklist"></a>App Service ellenőrzőlista
1. Tekintse át az App Service szakasz ebben a dokumentumban.
2. Hely biztonsági mentések kívül az elsődleges régióban karbantartása.
3. Részleges kimaradás esetén megkísérli lekérni az aktuális hely FTP-vel.
4. Tervezi a hely új vagy létező webhelyre egy másik régióban.
5. Plan configuration changes for both application and DNS CNAME records.

## <a name="hdinsight-checklist"></a>HDInsight feladatlista
1. Tekintse át a dokumentum a HDInsight szakaszát.
2. Hozzon létre egy új Hadoop-fürt a replikált adatok régió.

## <a name="sql-reporting-checklist"></a>Feladatlista SQL-alapú jelentések
1. Tekintse át az SQL Reporting szakaszt ebben a dokumentumban.
2. Egy másik SQL Reporting példányhoz egy másik régióban karbantartása.
3. A cél a régióba replikálja külön terv karbantartása.

## <a name="media-services-checklist"></a>A Media Services ellenőrzőlista
1. Tekintse át a dokumentum a Media Services szakaszát.
2. Media Services-fiók létrehozása egy másik régióban.
3. A két adatfolyam-továbbítási feladatátvételi támogatásához régió azonos tartalom kódolása.
4. A szolgáltatás szüneteltetése esetén egy másik régióban kódolási feladatok elküldéséhez.

## <a name="virtual-network-checklist"></a>Virtuális hálózati ellenőrzőlista
1. Tekintse át a virtuális hálózati szakasz ebben a dokumentumban.
2. Használja a virtuális hálózati beállításainak a hozza létre újból egy másik régióban exportált.

