---
title: 'Nagyszámítógépek migrálása: A nagyszámítógépes alkalmazások áttelepítése'
description: A nagyszámítógépes környezetek alkalmazásokat át az Azure-ba, a bevált, magas rendelkezésre állású és méretezhető infrastruktúrát Nagyszámítógépek a jelenleg futó rendszerek
author: njray
ms.date: 12/26/2018
ms.openlocfilehash: dcae5077e26ab8ba9b08e0da71a5e69d0d9f62e3
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899119"
---
# <a name="mainframe-application-migration"></a>A nagyszámítógépes alkalmazások áttelepítése

Környezetből nagyszámítógépes alkalmazások migrálása az Azure-ba, amikor a legtöbb csapatok kövesse a gyakorlati megközelítést: bárhol, és amikor csak lehetséges, és indítsa el a szakaszos bevezetéshez, ahol alkalmazások feladatátvitelt vagy cseréje.

Alkalmazás áttelepítés általában egy vagy több, az alábbi stratégiák foglalja magában:

- Áthelyezési: Meglévő kódot, programok és alkalmazások áthelyezését a nagygépes, és majd újra a kódot egy felhő-példányban tárolt nagyszámítógépes emulátorban való futtatásához. Ez a megközelítés általában egy felhőalapú emulátor alkalmazások való áthelyezésével, és ezután migrálás az adatbázis egy felhőalapú adatbázis kezdődik. Néhány mérnöki és újrabontás együtt az adat- és átalakítás szükségesek.

    Másik megoldásként is újratárolása, a hagyományos szolgáltató használatával. Egyik fő előnye az kihasználhatja a felhőben van kihelyezése infrastruktúra-felügyelettel. A nagyszámítógépes számítási feladatokat, üzemeltető adatközpont-szolgáltató található. Ez a modell előfordulhat, hogy vásárlás ideje, csökkentheti a szállító zárolási és előállítási közbenső költséget takaríthat meg.

- Vonja ki: Minden alkalmazás, amely már nincs szükség áttelepítés előtt kell vonni.

- Építse újra: Egyes szervezetek teljes mértékben a modern technikákkal programok újraírási válassza. Adja meg a hozzáadott járó bonyodalmak és költségek ezt a módszert, akkor sem, közös, a lift-and-shift-megközelítést. Az ilyen típusú áttelepítés után gyakran logikus kezdődik, és cserélje le a modulok és a kód átalakító motor használatával.

- Cserélje le ezt: Ez a megközelítés lecseréli nagyszámítógépes funkció hasonló szolgáltatásokat a felhőben. Szoftverszolgáltatás (SaaS) az egyik lehetőséget, amely egy vállalati szempont, például a Pénzügy, hr, gyártási vagy vállalati erőforrás-tervezés kifejezetten a létrehozott megoldást használ. Számos iparág-specifikus alkalmazások emellett problémáinak megoldásához korábban használt egyéni nagyszámítógépes megoldások, most érhetők el.

Kell érdemes úgy, hogy kezdetben a migrálni kívánt számítási feladatok megtervezése először, és ellenőrizze a kapcsolódó alkalmazás áthelyezése követelményekről, örökölt kódbázisaiba és -adatbázisok.

## <a name="mainframe-emulation-in-azure"></a>A nagyszámítógépes emulációs az Azure-ban

Az Azure cloud services emulációját hagyományos nagyszámítógépes környezetekben, így újból felhasználhatja a meglévő nagyszámítógépes kódot és az alkalmazások. Emulálhatja használt kiszolgáló-összetevők közé tartozik az online tranzakciófeldolgozási (OLTP), a batch és az adatok feldolgozási rendszerek.

### <a name="oltp-systems"></a>OLTP rendszerek

Számos Nagyszámítógépek rendelkezik, amely több ezer vagy több millió, a felhasználók nagy számú frissítések OLTP rendszerek. Ezek az alkalmazások gyakran használnak a tranzakció-feldolgozás és a képernyő-űrlap kezelési szoftvert, például a vásárlói adatokat verziókezelő rendszer (CICS), információ felügyeleti systes (IMS) és a Terminálszolgáltatások felület processzor (TIP).

OLTP-alkalmazások Azure-bA áthelyezésekor futtató infrastruktúra-szolgáltatás (IaaS) virtuális gépek (VM-EK) használatával az Azure-ban való emulátory Pro nagyszámítógépes tranzakció-feldolgozási (TP) figyelők érhetők el. A képernyő az állapotkezelés és az űrlap funkció webkiszolgálók által is meg lehet valósítani. Ez a módszer kombinálható adatbázis API-k, például az ActiveX-adatobjektum (ADO), nyissa meg az adatbázis-kapcsolat (ODBC) és a Java adatbázis-kapcsolat (JDBC) az adathozzáféréssel és -tranzakciókat.

### <a name="time-constrained-batch-updates"></a>Idő korlátozott batch-frissítések

Számos nagyszámítógépes rendszer felhasználóifiók-rekordok, mint például amelyek a kormányzati, banki és biztosítási milliónyi havi vagy éves frissítések végrehajtása. Nagyszámítógépek azzal kínál a nagy átviteli sebességű adatkezelési rendszerek ilyen típusú számítási feladatok kezelésére. Nagyszámítógépek kötegelt feladatok jellemzően soros jellegűek, és a bemeneti/kimeneti műveletek száma másodpercenként (IOPS) a teljesítmény a nagyszámítógépes gerinchálózatán által biztosított függenek.

Batch felhőalapú környezetek használata párhuzamos számítási és nagy sebességű hálózati teljesítményt. Kötegelt teljesítmény optimalizálása érdekében van szüksége, ha az Azure biztosít a különböző számítási, tárolási és hálózatkezelési lehetőségeket.

### <a name="data-ingestion-systems"></a>Adatok feldolgozási rendszerek

Nagyszámítógépek nagy váró adatokat kiskereskedelmi, pénzügyi, gyártási és más megoldások feldolgozásra képes feldolgozni. Az Azure-ban, használhat egyszerű parancssori segédprogrammal például [AzCopy](/azure/storage/common/storage-use-azcopy) céljából, illetve onnan tárolási helyét. Is használhatja a [Azure Data Factory](/azure/data-factory/introduction) szolgáltatás, lehetővé téve a létrehozása és ütemezése, adatvezérelt munkafolyamatok, különálló adattárakból származó adatokat.

Mellett emulációs környezetek, az Azure platform (PaaS) szolgáltatás és elemzési szolgáltatás, amely növelheti a meglévő nagyszámítógépes környezet.

## <a name="migrate-oltp-workloads-to-azure"></a>OLTP számítási feladatok migrálása az Azure-bA

A lift-and-shift-megközelítés a beállítás a nem kód a gyors áttelepítés meglévő alkalmazások Azure-bA. Egyes alkalmazások áttelepítése, amely biztosítja a kockázatokat a felhő előnyeinek vagy a kódmódosítások végrehajtásának költségeket. Ez a megközelítés egy emulatort használja nagyszámítógépes tranzakció-feldolgozási (TP) figyeli az Azure-ban támogatja.

TP figyelők érhetők el a különböző gyártók és a virtual machines szolgáltatásban, az infrastruktúrát, mint az Azure szolgáltatás (IaaS) lehetőséget. A következő előtt és után diagramok egy online alkalmazás IBM DB2, a relációsadatbázis-kezelő rendszer (DBMS), által támogatott áttelepítés nagygépes IBM z/OS megjelenítése. DB2 z/os virtuális tárolási hozzáférési módszer (VSAM) fájlok használatával tárolja az adatokat és az indexelt egymást követő hozzáférési módszer (ISAM) egybesimított fájlok. Ez az architektúra a tranzakció figyelés CICS is használ.

![Lift-and-shift az Azure-ban emulációs nagyszámítógépes környezet](../../_images/mainframe-migration/mainframe-vs-azure.png)

Az Azure-ban a TP kezelő és a batch-feladatok által használt JCL futtatásához használt emulációs környezetekben. Az adatszinten lévő DB2 helyébe [Azure SQL Database](/azure/sql-database/sql-database-technical-overview), bár a Microsoft SQL Server, a DB2 LUW vagy az Oracle Database is használható. Az emulátor támogatja a IMS VSAM és SEQ. A nagyszámítógépes felügyeleti eszközei Azure-szolgáltatások és más gyártók, virtuális gépeken futó szoftver helyébe lép.

A képernyő az állapotkezelés és az űrlap bejegyzés funkciók általában adatbázis API-k, például ODBC, JDBC és a ADO adatelérési és a tranzakciókért a webkiszolgálók segítségével van megvalósítva. Azure IaaS-összetevőt érdemes használni, a pontos line-up attól függ, hogy az operációs rendszer igény szerint. Példa:

- Windows-alapú virtuális gépek: Internet Information Server (IIS) képernyő kezelésére és üzleti logikát az ASP.NET együtt. Használja az ADO.NET adathozzáféréssel és -tranzakciókat.

- Linux-alapú virtuális gépek: A Java-alapú alkalmazáskiszolgálók elérhető, például az Apache Tomcat képernyő kezelésére és a Java-alapú üzleti funkció. Használjon JDBC adathozzáféréssel és -tranzakciókat.

## <a name="migrate-batch-workloads-to-azure"></a>A batch számítási feladatok migrálása az Azure-bA

Kötegelt műveletek az Azure-ban a tipikus batch környezet Nagyszámítógépek eltérnek. Nagyszámítógépes kötegelt feladatok általában soros jellegűek, és az IOPS, a teljesítmény a nagyszámítógépes gerinchálózatán által biztosított függenek. Batch felhőalapú környezetek párhuzamos számítási és a nagy sebességű hálózatok használata a teljesítmény.

Az Azure batch-teljesítmény optimalizálása érdekében fontolja meg a [számítási](/azure/virtual-machines/windows/overview), [tárolási](/azure/storage/blobs/storage-blobs-introduction), [hálózati](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/), és [figyelési](/azure/azure-monitor/overview) lehetőségek a következők szerint .

### <a name="compute"></a>Compute

Használat:

- A legmagasabb órajel rendelkező virtuális gépeket. Nagyszámítógépes alkalmazások vannak gyakran egyszálas és nagyszámítógépes processzor rendelkezik egy nagyon nagy órajel.

- Virtuális gépek nagy memóriaigényű kapacitással, hogy az adatok és alkalmazások gyorsítótárazását területeken működik.

- Virtuális gépek magasabb sűrűség Vcpu többszálas feldolgozás, ha az alkalmazás támogatja több szálon előnyeinek kihasználása érdekében.

- Párhuzamos feldolgozás, mivel Azure könnyen méretezhető párhuzamos feldolgozásra, további számítási alkalmazásfejlesztést futtatása a Batch.

### <a name="storage"></a>Storage

Használat:

- [Az Azure prémium szintű SSD](/azure/virtual-machines/windows/premium-storage) vagy [Azure Ultranagy SSD](/azure/virtual-machines/windows/disks-ultra-ssd) maximális rendelkezésre álló iops.

- Több lemezzel rendelkező a tároló mérete-nként több Percenkénti szétosztottsága befolyásolhatja.

- Tárolás az Azure storage több eszközön futó i/o particionálás.

### <a name="networking"></a>Hálózat

- Használat [Azure gyorsított hálózatkezelés](/azure/virtual-network/create-vm-accelerated-networking-powershell) késés minimalizálása érdekében.

### <a name="monitoring"></a>Figyelés

- Használja a monitorozási eszközökkel, [Azure Monitor](/azure/azure-monitor/overview), [Azure Application Insights](/azure/application-insights/app-insights-overview), és akár az Azure naplói lehetővé teszik a rendszergazdák bármely keresztül batch futtatások teljesítmény figyelésére, és segít kiküszöbölni a szűk keresztmetszeteket.

## <a name="migrate-development-environments"></a>Fejlesztési környezetek migrálása

Elosztott architektúrák a felhő fejlesztői eszközöket, amelyek az előnye, hogy a modern eljárások és programozási nyelveket külön készletét használják. Ez a váltás megkönnyítése érdekében, használhatja a fejlesztési környezet IBM z/OS-környezetek emulációjához tervezett egyéb eszközökkel. Az alábbi lista a Microsoft és más gyártók beállításai láthatók:

| Összetevő        | Azure-lehetőségek                                                                                                                                  |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| z/OS             | Windows, Linux vagy UNIX rendszerű                                                                                                                      |
| CICS             | Azure-szolgáltatások Micro fókusz, Oracle, GT Software (Fujitsu), TmaxSoft, Raincode és NTT adatok által kínált, vagy írja újra a Kubernetes használatával |
| IMS              | Azure-Micro fókusz és az Oracle által kínált szolgáltatások                                                                                  |
| Assembler –        | Azure-szolgáltatások Raincode és TmaxSoft; vagy COBOL, C, Java, vagy az operációs rendszer funkciók térkép               |
| JCL              | JCL, a PowerShell vagy egyéb parancsfájlkezelő eszközök                                                                                                   |
| COBOL            | COBOL, C, Java                                                                                                                            |
| Természetes          | Természetes, COBOL, C, Java                                                                                                                  |
| FORTRAN és PL / I | FORTRAN, PL / I, COBOL, C vagy Java                                                                                                           |
| REXX és PL / I    | REXX, a PowerShell vagy egyéb parancsfájlkezelő eszközök                                                                                                  |

## <a name="migrate-databases-and-data"></a>Adatbázisok és az adatok migrálása

Alkalmazás migrálását általában magában foglalja az adatréteg áthelyezését. Áttelepítheti az SQL Server, a nyílt forráskódú, és egyéb relációs adatbázisok, teljes körűen felügyelt megoldások az Azure-ban például [Azure SQL Database felügyelt példányába](/azure/sql-database/sql-database-managed-instance), [PostgreSQL-hez készült Azure Database Service](/azure/postgresql/overview), és [Azure Database for MySQL](/azure/mysql/overview) a [Azure Database Migration Service](/azure/dms/dms-overview).

Például ha a nagyszámítógépes adatrétegbeli használ telepíthet át:

- IBM DB2-höz vagy egy IMS adatbázis használata Azure SQL database, SQL Server, DB2 LUW vagy Oracle-adatbázishoz az Azure-ban.

- VSAM és egyéb egybesimított fájlok indexelt egymást követő hozzáférési módszer (ISAM) egybesimított fájlokat használja Azure SQL, az SQL Server, DB2 LUW vagy Oracle.

- Létrehozás dátuma csoportok (GDGs), egy elnevezési konvenciója és hasonló funkciókat nyújtanak, GDGs fájlnév-kiterjesztések használó fájlok az Azure-on át.

Az IBM adatréteg is át kell telepítenie néhány fő összetevőket tartalmazza. Például egy adatbázis áttelepítésekor is át szereplő adatok készletek, minden egyes tartalmazó dbextents, amelyek az operációs rendszer z/VSAM adatkészletek gyűjteménye. A migrálás tartalmaznia kell a címtár, amely azonosítja a tárolókészletet az adatok helye. A migrálási terv Ezenkívül figyelembe kell vennie az adatbázis naplója, amely tartalmaz egy rekordot az adatbázison végrehajtott műveletek. Egy adatbázis lehet egy, kettő (kettős vagy egy másik) vagy négy (kettős és alternatív) naplókat.

Adatbázis-migrálást is az alábbi összetevőket tartalmazza:

- Az adatbázis-kezelő: Az adatbázisban található adatok hozzáférést biztosít. Az adatbázis-kezelő saját partícióval z/OS-környezetben futtatható.

- Alkalmazás-kérelmező: Mielőtt továbbítanák azokat az alkalmazáskiszolgáló fogadja a alkalmazások kéréseit.

- Online erőforrás-adapter: Alkalmazás-kérelmező összetevők használatra CICS tranzakciók tartalmazza.

- Batch-erőforrás adapter: Kérelmező alkalmazásösszetevői valósítja meg az operációs rendszer z/a batch-alkalmazások.

- Interactive SQL (ISQL): Adja meg az SQL-utasítások vagy operátor parancsok CICS alkalmazás és felületet, így a felhasználók futtatások.

- CICS alkalmazás: Futtatások CICS, használja a rendelkezésre álló erőforrások és adatforrások CICS a ellenőrzése alatt.

- Batch-alkalmazás: Futtatások anélkül, hogy a felhasználók számára, például tömeges adatfrissítések előállítani, vagy jelentések készítése az adatbázisból interaktív kommunikációs logikai feldolgozni.

## <a name="optimize-scale-and-throughput-for-azure"></a>Méretezést és teljesítményt optimalizálása az Azure-hoz

Általánosan fogalmazva Nagyszámítógépek vertikális felskálázás, míg a felhőbeli elvégzi a horizontális felskálázást. Méretezést és teljesítményt az Azure-ban futó nagyszámítógépes stílusú alkalmazások optimalizálása érdekében fontos megérteni, hogyan Nagyszámítógépek külön és elkülönítse az alkalmazásokat. A – z/OS nagyszámítógépes nevű logikai partíció (LPARS) szolgáltatást használja a elkülönítése, és felügyelheti az erőforrásokat egy adott alkalmazáshoz egyetlen példányán.

Ha például egy nagyszámítógépes használható egy logikai partíciót (LPAR) egy CICS régióhoz tartozó COBOL programok és a egy külön LPAR DB2. További LPARs gyakran használják a fejlesztési, tesztelési és átmeneti környezeteket.

Az Azure-ban jelenleg egyre gyakoribb, erre a célra szolgál a különálló virtuális gépek használatával. Azure-architektúrákat általában az alkalmazás szintjén, az adatréteg, fejlesztéshez, egy másik készlet számára külön készlete virtuális gépek a virtuális gépek üzembe helyezése, és így tovább. Egyes rétegekbe tartozó feldolgozási ehhez a környezethez a legalkalmasabb típusú virtuális gépek és szolgáltatások használatával is lehet optimalizálni.

Emellett minden egyes réteghez is lehetővé teszi megfelelő vész helyreállítási szolgáltatások. Például éles és az adatbázis virtuális gépek szükség lehet a gyakori elérésű vagy online helyreállítás közben a fejlesztési és tesztelési virtuális gépeket támogatja a ritkán használt helyreállítási.

A következő ábrán látható egy lehetséges Azure üzembehelyezési egy elsődleges és másodlagos hely használatával. Az elsődleges helyen, az éles üzem előtti és tesztelési a virtuális gépek magas rendelkezésre állású üzembe. A másodlagos hely van a biztonsági mentésre és vészhelyreállításra.

![Egy lehetséges Azure üzembehelyezési egy elsődleges és másodlagos helyek használatával](../../_images/mainframe-migration/migration-backup-DR.png)

## <a name="perform-a-staged-mainframe-to-azure"></a>Hajtsa végre az Azure-bA a szakaszos nagyszámítógépes

Az Azure-bA a nagyszámítógépes megoldások áthelyezését is igénybe vehet egy *előkészített* , amellyel az egyes alkalmazások először helyezik és mások továbbra is a nagyszámítógépes ideiglenesen vagy véglegesen az áttelepítés. Ez a megközelítés általában szükséges a rendszerek, amelyek lehetővé teszik az alkalmazások és adatbázisok, a nagyszámítógépes és az Azure közötti együttműködését.

Gyakran előfordul, hogy egy alkalmazás az adatok a nagyszámítógépes a az alkalmazás által használt Azure-bA. Szoftver lehetővé teszik az alkalmazások az Azure-ban érheti el adatait a nagyszámítógépes szolgál. Szerencsére a megoldások széles skáláját adja meg az Azure és a meglévő nagyszámítógépes környezetek közötti integrációja, támogatja a hibrid forgatókönyvek kialakítását és migrálása idővel. Microsoft-partnerek, független szoftverszállítók és rendszerintegrátorok segítségével együttműködjünk.

Az egyik lehetőség van [Microsoft Host Integration Server](https://docs.microsoft.com/host-integration-server/) (a), a megoldás, amely az elosztott relációs adatbázis-architektúra (DRDA) szükséges hozzáférés az adatokhoz a DB2, amely továbbra is megtalálható a nagyszámítógépes alkalmazások az Azure-ban. A nagyszámítógépes – Azure-integráció más lehetőségek IBM, a az Attunity, Codit, más gyártók és a nyílt forráskódú lehetőségeket a megoldásokkal.

## <a name="partner-solutions"></a>Partneri megoldások

Ha a nagyszámítógépes migráció használatát fontolgatja, segítségére lehetnek a partneri ökoszisztéma érhető el.

Azure bevált, magas rendelkezésre állású és méretezhető infrastruktúrát biztosít a jelenleg futó Nagyszámítógépek rendszerekhez. Bizonyos munkaterhelések hitelesítésiházirend-telepíthető át. Más számítási feladatok ettől az korábbi rendszer szoftverek, például CICS és IMS, akkor is rehosted, partneri megoldások használatával, és az Azure-bA idővel át. Választott, függetlenül a Microsoft és partnerei számára elérhetők segítséget nyújt a nagyszámítógépes szoftver rendszerfunkcióit fenntartása mellett az Azure-ban optimalizálása.

Egy partneri megoldás kiválasztásával kapcsolatban részletes útmutatásért tekintse meg a [Platform korszerűsítése Alliance](https://www.platformmodernization.org/pages/mainframe.aspx).

## <a name="learn-more"></a>Részletek

További információkért lásd a következőket:

- [Bevezetés az Azure használatába](/azure)

- [Platform Modernization Alliance: Nagyszámítógépek migrálása](https://www.platformmodernization.org/pages/mainframe.aspx)

- [IBM DB2-höz pureScale Azure-beli üzembe helyezése](https://azure.microsoft.com/resources/deploy-ibm-db2-purescale-on-azure)

- [Host Integration Server (HIS) dokumentációja](https://docs.microsoft.com/host-integration-server/)
