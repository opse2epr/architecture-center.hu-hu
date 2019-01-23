---
title: Egy Azure-régióban, adatvesztés helyreállítása
description: Tudnivalók a tartalék rugalmas, magas rendelkezésre állású, hibatűrő alkalmazások tervezése, valamint a vészhelyreállítási.
author: adamglick
ms.date: 08/18/2016
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: 7f207bbc0bb0128126f9b828dc100d43553cb100
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487976"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Technikai útmutató az Azure rugalmassága: régióra kiterjedő szolgáltatáskimaradás helyreállítása

Azure oszlik fizikailag és logikailag egységek nevű régió. Egy régióban egy vagy több adatközpont hálózatbővítési áll.

Ritka esetekben is lehet, hogy egy teljes régió az eszközök, például hálózati hibák miatt elérhetetlenné válhat. Vagy létesítményekben elvesznek, teljes mértékben, például természeti katasztrófa miatt. Ez a szakasz ismerteti az Azure funkcióit régiók között elosztott alkalmazások létrehozásához. Az ilyen terjesztési segít minimálisra csökkentése érdekében, hogy egy adott régióban hiba hatással lehetnek más régióban lehetőségét.

## <a name="cloud-services"></a>Felhőszolgáltatások

### <a name="resource-management"></a>Erőforrás-kezelés

Számítási példányok régióban juttathatja el önálló felhőalapú szolgáltatásként létrehozásával minden célként megadott régióban, és majd tegye közzé a központi telepítési csomag minden felhőszolgáltatáshoz. Vegye azonban figyelembe, hogy forgalom elosztása különböző régiókban lévő felhőszolgáltatás kell megvalósítani, az alkalmazás fejlesztője vagy a forgalomkezelő szolgáltatásra.

Tartalék szerepkörpéldányok üzembe helyezéséhez előre vész-helyreállítási számának meghatározása a kapacitástervezés fontos eleme. A lehető legjobban hasonlítaniuk másodlagos telepítési kellene biztosítja, hogy kapacitás már érhető el, ha szükséges, azonban ez gyakorlatilag megduplázza a költségeket. Gyakori minta, hogy rendelkezik, amelyik kisebb, másodlagos, elég nagy kritikus szolgáltatások futtatásához. A kisméretű másodlagos környezetet, célszerű, mindkettő kapacitás, és a másodlagos környezet konfigurációjától tesztelési fenntartására.

> [!NOTE]
> Az előfizetési kvóta nem kapacitás garantálja. A kvóta egyszerűen kredit korlátozva. Garantálja a kapacitás, a szükséges számú szerepkörök definiálni kell a modell, és a szerepköröket kell telepíteni.

### <a name="load-balancing"></a>Terheléselosztás

Régiók közötti forgalom terheléselosztása megköveteli a egy adatforgalom-kezelő megoldás. Az Azure biztosít [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/). Emellett kihasználhatja a külső szolgáltatások, amelyek hasonló forgalom felügyeleti képességeket biztosítják.

### <a name="strategies"></a>Stratégiák

Számos alternatív stratégiák elosztott számítási megvalósításának régióban érhetők el. Ezek kell kialakítani, hogy az adott üzleti követelményeket és körülmények között az alkalmazás. Magas szinten a módszer a következő kategóriákba osztható:

- **A katasztrófa utáni ismételt üzembe helyezése**: Ebben a megközelítésben az alkalmazás rendszer újratelepítése előzmények vészhelyreállítási időpontjában. Ez a nem kritikus fontosságú a garantált helyreállítás ideje nem igénylő alkalmazások esetében.

- **Tartalék (aktív/passzív) meleg**: Egy másodlagos üzemeltetett szolgáltatás létrehozása egy másik régióban, és a szerepkörök telepítve. a minimális kapacitás; biztosítása a szerepkörök azonban éles forgalom nem kapja. Ez a megközelítés akkor lehet hasznos, nem a régiók közötti forgalom elosztását készített alkalmazások.

- **Gyakran használt adatok rétegére tartalék (aktív)**: Az alkalmazás éles terhelés fogadásához több régióban lett kialakítva. A cloud services, az egyes régiókban vészhelyreállítási célokból szükségesnél nagyobb kapacitást lehet konfigurálni. Másik lehetőségként a cloud services méretezése előfordulhat, hogy szükség esetén egy vészhelyreállítási és a feladatátvétel időpontjában ki. Ez a megközelítés alkalmazás kialakítása jelentős befektetéseket igényel, de ez jelentős előnyökkel jár. Ezek közé tartozik a kis- és garantált helyreállítási idő, a folyamatos tesztelés az összes helyreállítási helyeken, és a kapacitás hatékony kihasználását.

Elosztott tervezési teljes hatásának a megbeszélését ebben a dokumentumban hatókörén kívül esik. További információkért lásd: [vészhelyreállítás és magas rendelkezésre állás az Azure-alkalmazások](https://aka.ms/drtechguide).

## <a name="virtual-machines"></a>Virtual machines

A helyreállítási szolgáltatás (IaaS) virtuális gépeken (VM) az infrastruktúra hasonlít platform, a platformszolgáltatás (PaaS) számítási helyreállítási sok szempontból. Fontos funkciókülönbségek vannak, azonban oka, hogy az a virtuális gép, mind a Virtuálisgép-lemez áll egy IaaS-beli virtuális Gépen.

- **Régiók közötti, amelyek az alkalmazáskonzisztens biztonsági mentését az Azure Backup szolgáltatás használatával**.
  [Az Azure Backup](https://azure.microsoft.com/services/backup/) lehetővé teszi több virtuális gép lemezeinek alkalmazáskonzisztens biztonsági másolatok létrehozásához, és támogatja a replikációt a biztonsági mentések régióban az ügyfelek számára. Ehhez válassza a georeplikáció a mentési tároló létrehozásának időpontjában. Vegye figyelembe, hogy a replikáció, a biztonsági mentési tárolót is be kell állítani a létrehozásakor. Később állítható. Ha egy régió elveszett, Microsoft is elérhetővé a biztonsági másolatokat az ügyfelek számára. Ügyfelek tudják a beállított helyreállítási pontok visszaállítása.

- **Az adatlemez elkülönítése az operációsrendszer-lemez**. IaaS-beli virtuális gépek fontos szempont az, hogy az operációsrendszer-lemez nem módosítható a virtuális gép újbóli létrehozása nélkül. Ez nem probléma, ha a stratégia az, hogy katasztrófa utáni ismételt üzembe helyezése. Azonban lehet probléma a meleg tartalék megközelítés tartalékkapacitás való használatakor. Ennek megfelelően megvalósítása érdekében a megfelelő operációsrendszer-lemez, az elsődleges és másodlagos helyekre telepített kell rendelkeznie, és az alkalmazásadatok egy különálló meghajtón kell tárolni. Ha lehetséges használjon egy biztosíthatók a mindkét helyen szabványos operációs rendszer konfigurációját. A feladatátvétel után majd csatolja az adatmeghajtó a másodlagos tartományvezérlő a meglévő IaaS virtuális gépekhez. Az AzCopy használatával másolja az adatokat (eke) t, a pillanatképek egy távoli helyre.

- **Több Virtuálisgép-lemezek földrajzi feladatátvétel után konzisztencia kapcsolatos lehetséges problémák figyelembe**. Virtuálisgép-lemezek vannak implementálva az Azure Storage-blobokkal, és rendelkezik az azonos georeplikációs jellemző. Kivéve, ha [Azure Backup](https://azure.microsoft.com/services/backup/) van használt, garanciát nem jelentenek konzisztencia lemezeken, mert georeplikációs aszinkron, és egymástól függetlenül replikálja. Az egyes Virtuálisgép-lemezek garantáltan egy összeomlás-konzisztens az állapot egy földrajzi feladatátvétel után, de nem konzisztens lemezek között. Ez bizonyos esetekben (például esetén lemez csíkozást) problémákat okozhat.

## <a name="storage"></a>Tárhely

### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Helyreállítási blob, tábla, üzenetsor és a virtuális gép lemezes tárolás, Georedundáns tárolás használatával

Az Azure, blobok, táblák, üzenetsorok és a virtuális gép lemezei minden georeplikált alapértelmezés szerint. Erre hivatkozik, Georedundáns tárolást (GRS). GRS tároló adatait replikálja egy párosított adatközpontban több száz mérfölddel belül egy adott földrajzi régióban vannak. Adja meg a további tartóssága abban az esetben a fő adatközpont vészhelyreállítási, GRS célja. Feladatátvétel esetén a Microsoft vezérlők és a feladatátvétel súlyos vészhelyzetek esetére, amelyben az eredeti elsődleges helyre sikertelennek helyreállíthatatlan ésszerű időn belül korlátozódik. Bizonyos esetekben több napig is lehet. Az adatok általában replikációja néhány percen belül, bár a szinkronizálás időköze nem még jelez egy szolgáltatásiszint-szerződés.

Földrajzi feladatátvételt, ha ott nem lesz, hogyan érhető el, a fiók (az URL-cím és a fiókkulcsot nem változik). A storage-fiókban, azonban szerepelni fog egy másik régióban a feladatátvételt követően. Ez hatással lehet a storage-fiók regionális affinitás igénylő alkalmazásokhoz. Még a szolgáltatásokat és alkalmazásokat, amelyek nem igényelnek a storage-fiók ugyanabban az adatközpontban több adatközpontot késés és sávszélesség díj az előfordulhat, hogy lehet kényszerítő forgalom ideiglenesen áthelyezése a feladatátvételi régióban. Ez egy általános vész-helyreállítási stratégiát az sikerült tényező.

Mellett automatikus feladatátvétellel, GRS által biztosított Azure vezetett be egy szolgáltatás, amely olvasási hozzáférést biztosít a másodlagos tárolási helyen az adatok másolatát. Ezt az írásvédett Georedundáns tárolás (RA-GRS) nevezzük.

GRS- és RA-GRS tárolási kapcsolatos további információkért lásd: [Azure Storage replikáció](/azure/storage/storage-redundancy/).

### <a name="geo-replication-region-mappings"></a>Georeplikáció régióban leképezések

Fontos tudni, hogy hol található az adatok georeplikált, annak érdekében, hogy tudja, hol helyezi üzembe a többi példány regionális kapcsolatot azzal a storage igénylő adatait. További információ: [Azure párosított régiói](/azure/best-practices-availability-paired-regions).

### <a name="geo-replication-pricing"></a>Georeplikáció – díjszabás

Georeplikáció az Azure Storage jelenlegi díjszabás tartalmazza. Ezt nevezzük a Georedundáns Társzolgáltatási (GRS). Ha nem szeretné, hogy az adatok georeplikált georeplikációs letilthatja a fiókjához. Ezt nevezzük a helyileg redundáns tárolás, és a GRS képest kedvezményes díjért fel van töltve.

### <a name="determining-if-a-geo-failover-has-occurred"></a>Ha egy földrajzi feladatátvétel történt meghatározása

Földrajzi feladatátvételt akkor fordul elő, ha a program feladja a [Azure szolgáltatásállapot-irányítópult](https://azure.microsoft.com/status/). Alkalmazások valósíthat meg egy automatizált azt jelenti, hogy kívánja-e, ez azonban a tárfiók földrajzi régió-figyelési szolgáltatás által. Ez a más helyreállítási műveletek, például a számítási erőforrások, ahol azok átkerülnek a földrajzi régióban aktiválási indításához használható. Elvégezheti egy lekérdezést ehhez a service management API használatával [Tárfiók tulajdonságainak beolvasása](https://msdn.microsoft.com/library/ee460802.aspx). A releváns tulajdonságok a következők:

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>Virtuálisgép-lemezek és földrajzi feladatátvételt

Lásd a szakasz a Virtuálisgép-lemezek, garanciát nem jelentenek az adatkonzisztencia magyarázata a virtuális gép lemezeinek egy feladatátvétel után. Ahhoz, hogy a biztonsági mentések helyességét, egy biztonsági mentési termék például a Data Protection Manager biztonsági mentése és visszaállítása az alkalmazásadatok használandó.

## <a name="database"></a>Adatbázis

### <a name="sql-database"></a>SQL-adatbázis

Az Azure SQL Database két típusú helyreállítási biztosítja: A GEO-visszaállítás és aktív Georeplikációt.

#### <a name="geo-restore"></a>Georedundáns helyreállítás

[A GEO-visszaállítás](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) érhető el az alapszintű, Standard és prémium adatbázisok is. Ha az adatbázist üzemeltető régióban incidens miatt nem érhető el az adatbázis az alapértelmezett helyreállítási lehetőséget biztosít. Hasonló időponthoz visszaállítás, a Geo-visszaállítás adatbázisok biztonsági mentése az Azure georedundáns tárolási támaszkodik. A georeplikált biztonsági másolat a visszaállítást végzi, és ezért képes legyen ellenállni a az elsődleges régióban a storage leállásokat. További részletekért lásd: [visszaállítása egy Azure SQL Database vagy feladatátvétel a másodlagos](/azure/sql-database/sql-database-disaster-recovery/).

#### <a name="active-geo-replication"></a>Aktív georeplikáció

[Aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/) érhető el az összes adatbázis-szinten. A Geo-visszaállítás kínálnak agresszívabb helyreállítási követelményekkel rendelkező alkalmazások lett tervezve. Aktív Georeplikációt használ, a különböző régiókban lévő kiszolgálókon legfeljebb négy olvasható másodlagos példánnyal hozhat létre. Feladatátvétel a másodlagos adatbázisok bármelyikét is kezdeményezhető. Aktív Georeplikáció emellett támogatja az alkalmazás frissítése vagy adatáthelyezési forgatókönyv, valamint a terheléselosztás csak olvasható feladatokhoz használható. További információkért lásd: [Georeplikáció konfigurálása](/azure/sql-database/sql-database-geo-replication-portal/) és [átadja a feladatokat a másodlagos adatbázis](/azure/sql-database/sql-database-geo-replication-failover-portal/). Tekintse meg [aktív Georeplikáció használatával az SQL Database felhőalapú vészhelyreállítással alkalmazások tervezése](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) és [SQL adatbázis aktív Georeplikációthasználatávalafelhőalapúalkalmazásokműködésközbenifrissítésekkezelése](/azure/sql-database/sql-database-manage-application-rolling-upgrade/)a tervezzenek és valósítsanak meg, és alkalmazásokhoz való frissítés üzemkimaradás nélkül.

### <a name="sql-server-on-virtual-machines"></a>SQL Server on Virtual Machines

Számos lehetőség recovery és az SQL Server 2012 (és újabb) Azure Virtual Machines szolgáltatásban futó magas rendelkezésre állású érhetők el. További információkért lásd: [az SQL Server Azure virtuális gépek magas rendelkezésre állás és vészhelyreállítás helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="other-azure-platform-services"></a>Más Azure platformszolgáltatások

Amikor megpróbálja futtatni a cloud Services több Azure-régióban, meg kell fontolnia a következmények minden a függőségek. A következő szakaszokban a szolgáltatásspecifikus útmutató feltételezi, hogy egy másik Azure-adatközpontban kell használnia az azonos Azure-szolgáltatás. Ez magában foglalja a konfigurációs és az adatreplikáció feladatokat is.

> [!NOTE]
> Bizonyos esetekben ezeket a lépéseket segítségével csökkentheti a teljes adatközpontot esemény helyett a szolgáltatásspecifikus szolgáltatáskimaradás. Az alkalmazás szempontjából egy szolgáltatásspecifikus szolgáltatáskimaradás lehet igény szerint korlátozza és lenne szükség, a szolgáltatás átmenetileg áttelepíthető egy másodlagos Azure régióra.

### <a name="service-bus"></a>Service Bus

Az Azure Service Bus egy egyedi névtér, amely nem terjed ki az Azure-régiók használ. Ezért az első követelmény, állíthatja be a szükséges service bus-névterek a másodlagos régióban. Azonban szempontot is a tartósságot biztosítanak a várólistán lévő üzenetek. Többféle módon is üzenetek replikálásához az Azure-régiók között. Ezek a replikáció stratégiák és más vészhelyreállítási stratégiái részletekért lásd: [ajánlott eljárásai az alkalmazások a Service Bus leállásainak és katasztrófákkal szembeni szigetelő](/azure/service-bus-messaging/service-bus-outages-disasters/). Más rendelkezésre állási lehetőségekért lásd: [(elérhető). a Service Bus](recovery-local-failures.md#other-azure-platform-services).

### <a name="app-service"></a>App Service

Egy másodlagos Azure-régióba az Azure App Service-ben az alkalmazás, például a Web Apps és Mobile Apps, át, biztonsági másolatot a webhelyen érhető el közzétételre kell rendelkeznie. Ha a szolgáltatáskiesés megszüntetése után nem foglalja magában a teljes Azure-adatközpontban, esetleg FTP használatával töltse le a legutóbbi biztonsági másolata a webhely tartalmát. Ezután hozzon létre egy új alkalmazást a másodlagos régióban, kivéve, ha korábban ezt kapacitásának lefoglalása. A hely közzététele az új régióban, és végezze el a szükséges konfigurációs módosításokat. Ezek a változások lehetnek, adatbázis-kapcsolati karakterláncok vagy más régióban-specifikus beállításokat. Ha szükséges, a webhely SSL-tanúsítványt, és módosítsa a DNS CNAME-rekordot, hogy az egyéni tartománynevet az újratelepített Azure Web App URL-cím mutat.

### <a name="hdinsight"></a>HDInsight

Azure Blob Storage-ban alapértelmezés szerint a HDInsight társított adatokat tárolja. HDInsight megköveteli, hogy egy MapReduce-feladatok feldolgozásában, Hadoop-fürtöt kell elhelyezni, amely tartalmazza az elemzett adatok a storage-fiók ugyanabban a régióban. Használja az elérhető az Azure Storage georeplikációs szolgáltatás, a megadott férhet hozzá az adataihoz a másodlagos régió, ahol rendszer replikálta az adatokat, ha valamilyen okból az elsődleges régió már nem érhető el. A régióban, ahol az adatok replikálása, és feldolgozását, létrehozhat egy új Hadoop-fürtöt. Más rendelkezésre állási lehetőségekért lásd: [HDInsight (elérhető)](recovery-local-failures.md#other-azure-platform-services).

### <a name="sql-reporting"></a>SQL Reporting (SQL-jelentés)

Jelenleg egy Azure-régióban, adatvesztés utáni helyreállításhoz szükséges több SQL-példány különböző Azure-régióban. Ezek a példányok az SQL Reporting hozzá kell férnie az ugyanazokat az adatokat, és adatokat kell rendelkeznie a saját helyreállítási terv egy esetleges vészhelyzet esetén. Minden jelentés RDL-fájl külső biztonsági másolatait is fenntarthat.

### <a name="media-services"></a>Media Services

Az Azure Media Services rendelkezik egy másik helyreállítási módszert kódolás és streamelés céljából. Streamelési általában kritikus fontosságú egy regionális kimaradás során. Ehhez fel kell Media Services-fiók két különböző Azure-régióban. A kódolt tartalom mindkét régióban kell elhelyezni. Hiba esetén átirányíthatja a streamelési forgalmat a másodlagos régióba. Kódolás bármely Azure-régióban is elvégezhető. Ha kódolás időérzékeny, például az élő esemény feldolgozása során, kell állnia egy másodlagos adatközpont-feladatok elküldése a meghibásodások.

### <a name="virtual-network"></a>Virtuális hálózat

Konfigurációs fájlokat adja meg a leggyorsabb mód a állítsa be a virtuális hálózat egy másik Azure-régióban. Elsődleges Azure-régióban, a virtuális hálózat konfigurálása után [exportálhatja a virtuális hálózat beállításait](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) az adott hálózathoz hálózati konfigurációs fájlt. Az elsődleges régióban leállás [visszaállítani a virtuális hálózat](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) a tárolt konfigurációs fájlból. Ezután konfigurálja az egyéb felhőszolgáltatások, virtuális gépek vagy dolgozhat az új virtuális hálózatot létesítmények közötti beállításait.

## <a name="checklists-for-disaster-recovery"></a>Vész-helyreállítási Ellenőrzőlisták

## <a name="cloud-services-checklist"></a>Cloud Services ellenőrzőlista

1. Tekintse át a Cloud Services szakasz ebben a dokumentumban.
2. Hozzon létre egy régiók közötti vész-helyreállítási stratégiát.
3. Ismerje meg a másodlagos régióban kapacitás foglalása skálán.
4. Forgalom útválasztási eszközök, például az Azure Traffic Manager használata.

## <a name="virtual-machines-checklist"></a>Virtuális gépek ellenőrzőlista

1. Tekintse át a virtuális gépekre vonatkozó szakaszban ebben a dokumentumban.
2. Használat [Azure Backup](https://azure.microsoft.com/services/backup/) hozhat létre alkalmazáskonzisztens biztonsági másolatok régióban.

## <a name="storage-checklist"></a>Tárolási ellenőrzőlista

1. Tekintse át a Storage szakasz ebben a dokumentumban.
2. Ne tiltsa le a georeplikációs tárolási erőforrások.
3. Ismerje meg, a georeplikáció másodlagos régió feladatátvételkor.
4. Hozzon létre egyéni biztonsági stratégiák a felhasználó általi feladatátvételi stratégiát.

## <a name="sql-database-checklist"></a>Az SQL Database ellenőrzőlista

1. Tekintse át az SQL Database szakasz ebben a dokumentumban.
2. Használat [Geo-visszaállítás](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) vagy [Georeplikációs](/azure/sql-database/sql-database-geo-replication-overview/) megfelelő módon.

## <a name="sql-server-on-virtual-machines-checklist"></a>Az SQL Server virtuális gépek ellenőrzőlista használata

1. Tekintse át az SQL Server, a virtuális gépekre vonatkozó szakaszban ebben a dokumentumban.
2. Régiók közötti AlwaysOn rendelkezésre állási csoportok használata vagy az adatbázis-tükrözés.
3. Alternatív megoldásként biztonsági mentése és visszaállítása blob storage-bA.

## <a name="service-bus-checklist"></a>A Service Bus ellenőrzőlista

1. Tekintse át a Service Bus szakasz ebben a dokumentumban.
2. Konfigurálja a Service Bus-névtér egy másik régióban.
3. Fontolja meg az egyéni replikálási gyakorlata üzenetek régiók között elosztva.

## <a name="app-service-checklist"></a>Az App Service-ellenőrzőlista

1. Tekintse át az App Service-ben szakasz ebben a dokumentumban.
2. Biztosítja a webhely biztonsági mentések kívül az elsődleges régióba.
3. Részleges szolgáltatáskimaradás esetén megkísérli lekérni az aktuális hely FTP használatával.
4. Tervez üzembe helyezni a hely új vagy meglévő helyre egy másik régióban.
5. Tervezze meg a konfigurációs módosítások az alkalmazás és a DNS CNAME-rekordokat.

## <a name="hdinsight-checklist"></a>HDInsight-feladatlista

1. Tekintse át a HDInsight szakasz ebben a dokumentumban.
2. Hozzon létre egy új Hadoop-fürtöt a régióban, a replikált adatokkal.

## <a name="sql-reporting-checklist"></a>Az SQL jelentési ellenőrzőlistát

1. Tekintse át az SQL Reporting szakaszt ebben a dokumentumban.
2. Egy másik SQL Reporting példányhoz egy másik régióban lévő karbantartása.
3. Replikálni a célként megadott régióban egy különálló csomag fenntartásához.

## <a name="media-services-checklist"></a>A Media Services ellenőrzőlista

1. Tekintse át a Media Services szakasz ebben a dokumentumban.
2. Hozzon létre egy Media Services-fiók egy másik régióban.
3. A kódolás kimenete streamelési feladatátvétel támogatásához mindkét régióban ugyanahhoz a tartalomhoz.
4. Egy szolgáltatás bekövetkező szolgáltatáskimaradás esetén egy másodlagos régióba kódolási feladatok elküldéséhez.

## <a name="virtual-network-checklist"></a>Virtuális hálózat ellenőrzőlista

1. Tekintse át a virtuális hálózat szakasz ebben a dokumentumban.
2. Használja újra létre egy másik régióban található virtuális hálózati beállítások exportálása.
