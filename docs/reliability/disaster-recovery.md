---
title: Hiba és a katasztrófa utáni helyreállítás az Azure-alkalmazásokhoz
description: Az Azure-ban megközelíti a vész-helyreállítási áttekintése
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: f00341b06b8092c798d6260317226190cbace66b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646556"
---
# <a name="failure-and-disaster-recovery-for-azure-applications"></a>Hiba és a katasztrófa utáni helyreállítás az Azure-alkalmazásokhoz

*Vész-helyreállítási* az alkalmazás funkciói egy végzetes elvesztése nyomán visszaállítása során a rendszer.

A korlátozott funkciókkal szintű katasztrófa során egy üzleti döntés, amely, egyik alkalmazásból a következőre változik. Ez lehet, hogy egyes alkalmazások teljesen nem érhető el, vagy részlegesen elérhetők az korlátozott funkciókkal elfogadható, vagy feldolgozási egy ideig késik. Más alkalmazások bármilyen csökkentett szolgáltatáskészletű nem fogadható el.

## <a name="disaster-recovery-plan"></a>Vész-helyreállítási terv

Először hozzon létre egy helyreállítási tervbe. A terv akkor tekinthető után azt teljes körűen tesztelve lett. A személyeket, folyamatokat és szolgáltatásokat biztosít az ügyfelek számára definiált szolgáltatásiszint-szerződés (SLA) a visszaállításához szükséges alkalmazások közé tartozik.

Vegye figyelembe az alábbi javaslatokat, amikor létrehozása és tesztelése a vész-helyreállítási terv:

- A csomag tartalmazzák a folyamat ügyfélszolgálaton és a problémák továbbítása a problémákat. Ezt az információt, elkerülheti a hosszan tartó, először nyerhető ki a helyreállítási folyamat segítségével.
- Értékelje az alkalmazáshibák üzletmenetre gyakorolt hatását.
- Válassza ki a régiók közötti recovery architektúrájáról, alapvető fontosságú alkalmazásokhoz.
- A konkrét tulajdonost, a vész-helyreállítási terv, beleértve az automatizálási és tesztelési azonosításához.
- A dokumentum a folyamat, különösen összes manuális lépést.
- A folyamat a lehető legnagyobb mértékben automatizálja.
- Rendszeresen meghatározzák az összes hivatkozást és a tranzakciós adatok, és a teszt biztonsági mentési visszaállítani egy biztonsági mentési stratégia.
- Riasztásokat állíthat be az Azure-szolgáltatást az alkalmazás által felhasznált verem.
- A terv kezelői létszámmal betanításához.
- Hajtsa végre a rendszeres vészhelyreállítási szimulációk miként ellenőrzi és javítja a tervet.

Ha használ [Azure Site Recovery](/azure/site-recovery/) replikálni a virtuális gépek (VM), a teljes alkalmazás feladatátvételét teljesen automatizált helyreállítási terv létrehozásához.

## <a name="manual-responses"></a>Manuális válaszok

Automation ideális megoldást biztosít, néhány vész-helyreállítási stratégiát manuális válaszok igényelnek.

### <a name="alerts"></a>Riasztások

Kísérje figyelemmel az alkalmazása olyan figyelmeztető jeleit, amelyek proaktív beavatkozást igényelhetnek. Például ha az Azure SQL Database vagy Azure Cosmos DB rendszeresen szabályozza az alkalmazását, szüksége lehet az adatbázis-kapacitását növelni, vagy a lekérdezések optimalizálásához. Annak ellenére, hogy az alkalmazás átlátható módon kezelni a szabályozási hibákat, a telemetria kell továbbra is riasztást küld, így nyomon lehet követni.

A szolgáltatási korlátai és kvóták küszöbértékeit javasoljuk, hogy riasztások konfigurálása az Azure-erőforrások metrikák és diagnosztikai naplókat. Ha lehetséges, állítsa be a riasztások a metrikák, amelyek kisebb késést biztosít, mint a diagnosztikai naplók.

Keresztül [Resource Health](/azure/service-health/resource-health-checks-resource-types), az Azure biztosít néhány beépített állapot, amely segít diagnosztizálni a szabályozási problémák az Azure szolgáltatás állapotát ellenőrzi.

### <a name="failover"></a>Feladatátvétel

A vész-helyreállítási stratégiát minden egyes Azure-alkalmazás és az Azure-szolgáltatások konfigurálása. Az SLA-k minden egyes alkalmazásokhoz szükség lehet, hogy megoldástól elfogadható központi telepítési stratégiát vész-helyreállítási támogatásához.  

Az Azure biztosít sok Azure-szolgáltatások lehetővé teszik a manuális feladatátvételt, például a különböző szolgáltatásokat [redis cache geo-replikák](/azure/azure-cache-for-redis/cache-how-to-geo-replication), vagy az automatikus feladatátvételt, például [SQL automatikus feladatátvételi csoportok](/azure/sql-database/sql-database-auto-failover-group). Példa:

- Az alkalmazás, amely főként használja a virtuális gépek használhatja az Azure Site Recovery a web- és logikai rétege számára. További információkért lásd: [Azure-bA vész-helyreállítási architektúra](/azure/site-recovery/azure-to-azure-architecture). Az SQL Server virtuális gépeken, használjon [SQL Server Always On rendelkezésre állási csoportok](/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-dr).
- Az App Service-ben és az Azure SQL Database-t használó alkalmazás használhat egy kisebb szintű App Service-csomag konfigurálása a másodlagos régióban, amely elvégzi az automatikus skálázást egy feladatátvétel esetén. Feladatátvételi csoportok használata az adatbázisszint számára.

Mindkét esetben egy [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) profilt biztosít az automatikus feladatátvételi régióban. [Terheléselosztók](/azure/load-balancer/load-balancer-overview) vagy [az application Gateway átjárók](/azure/application-gateway/overview) kell beállítani a másodlagos régió támogatja a feladatátvételt követően gyorsabb rendelkezésre állást.

### <a name="operational-readiness-testing"></a>Működési készenlét tesztelése

Tesztelje a működési készenlétet hajtsa végre, a másodlagos régióba történő feladatátvételt és feladat-visszavétel az elsődleges régióba. Sok Azure-szolgáltatás támogatja a manuális feladatátvételt vagy a feladatátvételi teszteket vészhelyreállítási próbák végrehajtása céljából. Azt is megteheti szimulálhatja kimaradás leállítása, vagy Azure-szolgáltatás eltávolításával.

## <a name="application-failure"></a>Alkalmazáshiba

Alkalmazáshibák vagy a helyreállítható, vagy a következő fájlnál is. Csökkentheti a Helyrehozható hiba, de a következő fájlnál hibák tartalomtérkép érhető el az alkalmazást.

- Bizonyos hibák automatikusan hibák kezeléséhez, és más műveleteket, átlátható módon lehet megoldani. Például a Traffic Manager automatikusan kezeli a gazdagép virtuális gép a mögöttes hardver vagy operációs rendszer szoftvert eredő hibák.
- Bizonyos hibák esetén az alkalmazás továbbra is korlátozott funkciókkal rendelkező felhasználói kérelmek kezelését.
- Súlyosabb szolgáltatáskimaradás lehet, hogy jelennek meg az alkalmazás nem érhető el.

Egy jól megtervezett rendszer elkülöníti a szolgáltatási szintű feladatokról &mdash; tervezési időben, és futásidőben. Ez a fajta elkülönítés megakadályozza, hogy egy függő szolgáltatáskimaradás lép fel, a teljes alkalmazás leáll. Vegyük példaként a következő modulokat a kereskedelmi webalkalmazás:

![A LEGÚJABB HIVATKOZÁS HOZZÁADÁSA](./_images/disaster-recovery.png)

Ha az adatbázis üzemeltetéséhez a rendelések leáll, a rendelések feldolgozása szolgáltatás értékesítési tranzakció nem tudja feldolgozni. Attól függően, az architektúra nem lehetséges a rendelés benyújtása és a rendelések feldolgozása szolgáltatások továbbra is érdemes lehet. Ha viszont termék adata egy másik helyen található, a termékkatalógus, továbbra is elérhető, bár előfordulhat, hogy az alkalmazás egyéb részei nem használhatók.

Van arra, hogy határozza meg, hogyan az alkalmazás fogja tájékoztatni a felhasználókat olyan átmeneti probléma, és hozhat létre a megfelelő értesítéseket a rendszer. Az előző példában az alkalmazás megtekintése a termékek és a egy bevásárlókosárhoz hozzáadását lehetővé teheti. Azonban amikor az ügyfél megpróbál e vásárolni valamit, az alkalmazás értesítenie kell őket, hogy a rendelési funkciók átmenetileg nem érhető el. Bár az ügyfél nem ideális, ez a módszer megakadályozza, hogy egy alkalmazás kiterjedő szolgáltatáskimaradás.

## <a name="data-corruption-and-restoration"></a>Az adatok sérülésével és visszaállítás

Ha egy adattár meghibásodik, előfordulhat, adatinkonzisztencia mikor válik elérhetővé, különösen akkor, ha a rendszer replikálta az adatokat. A helyreállítási időre vonatkozó célkitűzés (RTO) és a helyreállítási pont időkorlát (RPO) a replikált adatok készleteinek segít előre jelezni az adatveszteség mennyisége ismertetése.

Szeretné megtudni, hogy manuálisan vagy a Microsoft elindult-e a a régiók közti feladatátvétel, tekintse át az Azure-szolgáltatás SLA-k. Nincs SLA-k szolgáltatások régiók közti feladatátvétel, a Microsoft általában úgy dönt, hogy ha a feladatátvételt, és általában priorizálja az elsődleges régióban adatok helyreállítását. Ha az elsődleges régióban lévő adatok helyreállíthatatlanná sikertelennek, Microsoft átadja a feladatokat a másodlagos régióba.

### <a name="restoring-data-from-backups"></a>Adatok biztonsági másolatokból való visszaállítással

Biztonsági másolatok megóvja a véletlen törlés és adatok sérülése miatt az alkalmazás részét képező elvesztése. Az összetevő egy korábbi időpontra, amelyek használatával állítsa vissza a egy működési verziója megőrizni azokat.

Vészhelyreállítási stratégiák nem helyettesíti a biztonsági mentések, de az alkalmazásadatok rendszeres biztonsági mentést támogatja az egyes vész-helyreállítási helyzetekben. A biztonsági mentési tár választási lehetőségek vész-helyreállítási stratégiát kell alapulnia.

A biztonsági mentési folyamat futtatásának gyakoriságát határozza meg a helyreállítási Időkorlát. Például ha katasztrófa történik a biztonsági mentés előtt két perc, óránként végez biztonsági mentéseket, 58 percnyi adat elveszti. A vész-helyreállítási terv tartalmaznia kell, hogyan foglalkozni fog elveszett adatok.

Szokás a referenciaadatok egy másik tárolóban egy adattárban. Vegyük példaként SQL-adatbázis, az oszlop az Azure Storage-blob mutat. Biztonsági mentések nem egyszerre következnek be, ha az adatbázis lehet egy blobba, nem biztonsági másolatot a hiba előtt mutató. Az alkalmazás vagy a vész-helyreállítási terv musí implementovat rozhraní folyamatok kezeléséhez a inkonzisztenciát helyreállítás után.

> [!NOTE]
> Egyes forgatókönyvekben, például a virtuális gépek biztonsági mentése segítségével, amely [Azure Backup](/azure/backup/backup-azure-vms-first-look-arm), visszaállíthatja az csak egy biztonsági másolatból ugyanabban a régióban. Más Azure-szolgáltatások, például [Azure Cache redis](/azure/azure-cache-for-redis/cache-how-to-geo-replication), adja meg a georeplikált biztonsági mentéseit támogatják, segítségével helyreállítását végző szolgáltatásokat régiók között elosztva.

### <a name="azure-storage-and-azure-sql-database"></a>Az Azure Storage és az Azure SQL Database

Az Azure automatikusan tárolja az Azure Storage és SQL Database adatok különböző tartalék tartományban belül háromszor ugyanabban a régióban. Ha georeplikációt használ, az adatok további három alkalommal tárolódik egy másik régióban. Azonban ha az adatok sérült vagy törölni az elsődleges verziót (például felhasználói hiba) miatt, a módosításokat replikálja a többi példányt.

A lehetséges adatsérülés vagy törlés kezelésére két lehetősége van:

- **Hozzon létre egy egyéni biztonsági mentési stratégia.** A biztonsági másolatok tárolása Azure-ban vagy a helyszínen, attól függően, az üzleti követelmények és a szabályozási előírásoknak.
- **Használja a-időponthoz visszaállítási lehetőséget** SQL-adatbázis helyreállításához.

#### <a name="azure-storage-recovery"></a>Az Azure Storage-helyreállítás

Egy egyéni biztonsági mentési folyamat fejlesztés az Azure Storage, vagy használja számos külső biztonsági mentési eszközök egyikét.

Az Azure Storage biztosítja az adatok rugalmasságát biztosítja az automatikus replikációval, de ez nem akadályozza alkalmazáskód vagy a felhasználók károsíthassák az adatokat. Adatokat pontosságú alkalmazás vagy felhasználó hiba után szükséges fejlett technikák, például az adatok másolásának auditálási naplóba kerülnek a másodlagos tárolóhelyre. Több lehetősége van:

- [**Blokkblobok.**](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) Minden egyes blokkblob időponthoz pillanatkép létrehozása. Minden pillanatkép díjkötelesek csak óta a korábbi pillanatkép-állapotot a különbségek a blobon belüli tárolásához szükséges tárhelyet. A pillanatképek függnek az eredeti blob, ezért azt javasoljuk, másolás és a egy másik blob vagy akár egy másik tárfiókba. Ez a megközelítés biztosítja, hogy biztonsági mentési adatok véletlen törlés elleni védelme. Használat [AzCopy](/azure/storage/common/storage-use-azcopy) vagy [Azure PowerShell-lel](/azure/storage/common/storage-powershell-guide-full) a blobok másolása egy másik tárfiókba.

    További információkért lásd: [létrehozása egy pillanatképet egy Blobról](/rest/api/storageservices/creating-a-snapshot-of-a-blob).

- [**Azure Files.**](/azure/storage/files/storage-files-introduction) Használat [megosztási pillanatképek](/azure/storage/files/storage-snapshots-files), az AzCopy, vagy a PowerShell a fájlok másolása egy másik tárfiókba.
- [**Az Azure Table storage.**](/azure/storage/tables/table-storage-overview) Az AzCopy segítségével egy másik régióban egy másik tárfiókba történő táblák adatainak exportálása.

#### <a name="sql-database-recovery"></a>SQL-adatbázis-helyreállítás

Az üzleti szembeni az adatvesztést, SQL Database automatikusan végrehajtja az adatbázis teljes biztonsági mentését hetente, a különbözeti adatbázis biztonsági mentését óránként, és a tranzakciós biztonsági mentések minden 5 10 perc. A Basic, Standard és prémium szintű SQL Database szinten időponthoz visszaállítás, visszaállítás adatbázist használnak egy korábbi időpontra. További információ a következő cikkekben talál:

- [Automatikus biztonsági adatbázismentés használatával Azure SQL-adatbázis helyreállítása](/azure/sql-database/sql-database-recovery-using-backups)
- [Az Azure SQL Database üzletmenet-folytonossági funkcióinak áttekintése](/azure/sql-database/sql-database-business-continuity)

Egy másik lehetőség, az aktív georeplikáció által használandó SQL-adatbázis, amely az adatbázis-módosítások automatikusan replikálja a másodlagos adatbázisok azonos vagy eltérő Azure-régióban. További információkért lásd: [létrehozása és az aktív georeplikáció használatával](/azure/sql-database/sql-database-active-geo-replication).

Is több manuális módszer használata a biztonsági mentés és visszaállítás:

- Használja a **adatbázis-másolat** az adatbázis biztonsági másolatának létrehozása tranzakció-konzisztencia a parancsot.
- Az Azure SQL Database Import/Export Service, exportáló adatbázisok BACPAC fájlok (az adatbázisséma és a társított adatokat tartalmazó tömörített fájlok), az Azure Blob storage-ban tárolt támogató használja. Régióra kiterjedő szolgáltatáskimaradás ellen, a BACPAC-fájlok másolása egy másik régióba.

### <a name="sql-server-on-vms"></a>Az SQL Server virtuális gépeken

A virtuális gépeken futó SQL Server, két lehetősége van: a hagyományos biztonsági mentéseket és a naplóküldésben.

- A hagyományos biztonsági másolatokkal időben egy adott időpontra is visszaállíthatók, de a helyreállítási folyamat lassú. A hagyományos biztonsági mentések visszaállítása szükséges, hogy egy kezdeti biztonsági mentés kezdődhet, és ezután alkalmazza az összes növekményes biztonsági mentést.
- Beállíthatja, hogy a napló szállítási munkamenet a Naplók biztonsági másolatainak visszaállítás késleltetése. Ez lehetővé teszi a hibák az elsődleges replikán végrehajtott ablakot.

### <a name="azure-database-for-mysql-and-azure-database-for-postgresql"></a>Azure Database for MySQL és az Azure Database for postgresql-hez

Az Azure Database for MySQL és Azure Database for PostgreSQL az adatbázis-szolgáltatás automatikusan lehetővé teszi a biztonsági mentés öt percenként. Ezeket az automatikus biztonsági mentések segítségével visszaállítása a kiszolgáló és a hozzá tartozó adatbázisok egy korábbi időpontra az új kiszolgálóra. További információkért lásd:

- [Hogyan biztonsági mentése és visszaállítása egy kiszolgálót az Azure Database for MySQL-hez az Azure portal használatával](/azure/mysql/howto-restore-server-portal)
- [Hogyan biztonsági mentése és visszaállítása egy kiszolgálót az Azure Database for postgresql-hez az Azure portal használatával](/azure/postgresql/howto-restore-server-portal)

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

A cosmos DB rendszeres időközönként automatikusan lehetővé teszi a biztonsági mentés. Biztonsági mentés egy másik storage szolgáltatásban tárolódnak, és globálisan replikálva vannak regionális katasztrófák elleni védelem érdekében. Ha véletlenül törli az adatbázis vagy -gyűjteményben, küldjön egy támogatási jegyet, vagy hívja a legutóbbi automatikus biztonsági mentés visszaállíthatja az adatokat az Azure ügyfélszolgálatától. További információkért lásd: [Online biztonsági mentés és az Azure Cosmos DB igény szerinti visszaállítási](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-virtual-machines"></a>Azure-alapú virtuális gépek

Azure Virtual Machines alkalmazáshibák vagy véletlen törlés elleni védelem, használjon [Azure Backup](/azure/backup/). A létrehozott biztonsági mentéseket egységesek több Virtuálisgép-lemezek. Emellett az Azure Backup-tárolóban is lehet régiók közötti replikálására, egy regionális elmaradásából helyreállítás támogatásához.

## <a name="network-outage"></a>Hálózati kimaradás

Ha az Azure-hálózat részét nem érhető el, nem feltétlenül tudni elérni az alkalmazást vagy adatokat. Ebben az esetben javasoljuk, hogy a vész-helyreállítási stratégiát a legtöbb alkalmazást futtathatják az korlátozott funkciókkal tervezése.

Ha csökkenti a funkciót nem lehet beállítani, a fennmaradó beállításokat alkalmazás állásidő és a egy másodlagos régióba történő feladatátvételt.

A korlátozott funkciókkal forgatókönyvek esetében:

- Ha az alkalmazás egy Azure-beli hálózati szolgáltatáskimaradás miatt nem érhető el az adatok, lehet, helyi futtatásához csökkentett az alkalmazás funkciói a gyorsítótárazott adatok használatával.
- Előfordulhat, hogy egy másik helyen adatok tárolására, amíg a kapcsolat helyreáll.

## <a name="dependent-service-failure"></a>Függő szolgáltatások hibája

Az egyes függő szolgáltatások tisztában kell lennie egy szolgáltatáskimaradás lép fel, és reagáljon az alkalmazás ugyanúgy következményekkel járhat. Számos szolgáltatás többek között, amelyek támogatják a rugalmasság és a rendelkezésre állás, így egymástól függetlenül kiértékelése a minden szolgáltatás valószínűleg javítható a vészhelyreállítási tervet. Például az Azure Event Hubs támogatja [feladatátvétele](/azure/event-hubs/event-hubs-geo-dr#setup-and-failover-flow) a másodlagos névtérre.

## <a name="region-wide-service-disruptions"></a>Régióra kiterjedő szolgáltatáskimaradás

Számos hiba kezelhető ugyanazon Azure-régióban. Azonban a régióra kiterjedő szolgáltatáskimaradás nem valószínű esetben a helyileg redundáns másolatait az adatok nem érhetők el. Ha engedélyezte a georeplikáció, nincsenek a blobok és a egy másik régióban lévő táblák három további másolatot készít. Ha a Microsoft deklarálja a régió elveszett, az Azure ismételt leképezés a másodlagos régióba DNS-bejegyzéseket.

> [!NOTE]
> Ez a folyamat csak régióra kiterjedő szolgáltatáskimaradás következik be, és nem esik a vezérlő. Fontolja meg [Azure Site Recovery](/azure/site-recovery/) jobb RPO és RTO érhető el. A Site Recovery segítségével eldöntheti, mi elfogadható kimaradás, és mikor kell a replikált virtuális gépek a feladatátvételt.

A válasz egy régióra kiterjedő szolgáltatáskimaradás attól függ, az üzembe helyezés és a vészhelyreállítási tervet.

- Költségvezérlés stratégiánk, nem kritikus fontosságú alkalmazásokhoz, amelyek nem igényelnek a garantált helyreállítás idejét célszerű az ismételt üzembe helyezése egy másik régióba.
- Az alkalmazásokat, amelyek egy másik régióban üzembe helyezett szerepkörökkel vannak tárolva, de nem osztja el a forgalmat több régió (*aktív/passzív telepítési*), váltson át a másodlagos üzemeltetett szolgáltatás a másodlagos régióban.
- Az alkalmazásokat, amelyek a lehető legjobban hasonlítaniuk másodlagos központi telepítéssel rendelkezik, egy másik régióban (*aktív/aktív központi telepítés*), irányíthatja a forgalmat az adott régióban.

Régióra kiterjedő szolgáltatáskimaradás utáni kapcsolatos további információkért lásd: [régióra kiterjedő szolgáltatáskimaradás helyreállítása](../resiliency/recovery-loss-azure-region.md).

### <a name="vm-recovery"></a>Virtuális gép helyreállítási

A kritikus fontosságú alkalmazások esetén tervezze meg a virtuális gépek helyreállítása régióra kiterjedő szolgáltatáskimaradás esetén.

- Használja az Azure Backup vagy egy másik biztonsági mentési módszer a régiók közötti, amelyek az alkalmazáskonzisztens biztonsági mentések létrehozását. (A Backup-tároló replikálását is be kell állítani a létrehozásakor.)
- A Site Recovery használatával replikálja az alkalmazás egykattintásos feladatátvétel és a feladatátvétel tesztelése régióban.
- A Traffic Manager segítségével automatizálhatja a felhasználói forgalom feladatátvételének egy másik régióban.

További tudnivalókért lásd: [egy régióra kiterjedő szolgáltatáskimaradás lép fel, a virtuális gépek helyreállítása](../resiliency/recovery-loss-azure-region.md#virtual-machines).

### <a name="storage-recovery"></a>Storage-helyreállítás

A tároló védelmére régióra kiterjedő szolgáltatáskimaradás esetén:

- Georedundáns tárolás használata.
- Tudja, ahol a tárolási georeplikált. Ez hatással van, ahol az adatok regionális kapcsolatot azzal a storage igénylő többi példánya telepít.
- Ellenőrizze az adatokat a konzisztencia feladatátvétel után, és ha szükséges, állítsa vissza biztonsági másolatból.

További tudnivalókért lásd: [RA-GRS használatával magas rendelkezésre állású alkalmazások tervezése](/azure/storage/common/storage-designing-ha-apps-with-ragrs).

### <a name="sql-database-and-sql-server"></a>Az SQL Database és SQL Server

Az Azure SQL Database két típusú helyreállítási biztosítja:

- A geo-visszaállítás használatával állítsa vissza egy adatbázist a biztonsági másolat egy másik régióban. További információkért lásd: [helyreállítani egy Azure SQL database automatikus biztonsági adatbázismentés használatával](/azure/sql-database/sql-database-recovery-using-backups).
- Aktív georeplikáció használatával átadja a feladatokat egy másodlagos adatbázist. További információkért lásd: [létrehozása és az aktív georeplikáció használatával](/azure/sql-database/sql-database-active-geo-replication).

Tekintse meg a virtuális gépeken futó SQL Serverhez [az SQL Server Azure virtuális gépek magas rendelkezésre állás és vészhelyreállítás helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

## <a name="service-specific-guidance"></a>Szolgáltatásspecifikus útmutató

Az alábbi cikkek ismertetik a vészhelyreállítást az adott Azure-szolgáltatásokhoz:

| Szolgáltatás                       | Cikk                                                                                                                                                                                       |
|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Azure Database for MySQL      | [Az Azure Database for MySQL üzletmenet-folytonossági funkcióinak áttekintése](/azure/mysql/concepts-business-continuity)                                                  |
| Azure Database for PostgreSQL | [Az Azure Database for PostgreSQL üzletmenet-folytonossági funkcióinak áttekintése](/azure/postgresql/concepts-business-continuity)                                        |
| Azure Cloud Services          | [Mi a teendő az Azure Cloud Servicest befolyásoló Azure szolgáltatás kiesése esetén?](/azure/cloud-services/cloud-services-disaster-recovery-guidance) |
| Cosmos DB                     | [Az Azure Cosmos DB magas rendelkezésre állás](/azure/cosmos-db/high-availability)                                                                                |
| Azure Key Vault               | [Az Azure Key Vault rendelkezésre állás és redundancia](/azure/key-vault/key-vault-disaster-recovery-guidance)                                                        |
| Azure Storage                 | [Katasztrófa utáni helyreállítás és a tárolási fiók feladatátvételi (előzetes verzió) az Azure Storage-ban](/azure/storage/common/storage-disaster-recovery-guidance)                       |
| SQL Database                  | [Egy Azure SQL Database visszaállítása vagy feladatátvétel egy másodlagos régióba](/azure/sql-database/sql-database-disaster-recovery)                                       |
| Virtuális gépek              | [Mi a esetén az Azure szolgáltatás megszakadása hatással van az Azure-felhőben](/azure/cloud-services/cloud-services-disaster-recovery-guidance)               |
| Azure Virtual Network         | [Virtuális hálózat – üzletmenet-folytonossági](/azure/virtual-network/virtual-network-disaster-recovery-guidance)                                                  |

## <a name="next-steps"></a>További lépések

- [Helyreállítás adatsérülés vagy véletlen törlés](../resiliency/recovery-data-corruption.md)
- [Régióra kiterjedő szolgáltatáskimaradás helyreállítása](../resiliency/recovery-loss-azure-region.md)