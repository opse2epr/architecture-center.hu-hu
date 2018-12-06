---
title: Az Azure-alkalmazások vészhelyreállítása
description: Technikai áttekintése és részletes információk a vész-helyreállítási a Microsoft Azure-alkalmazások tervezéséhez.
author: adamglick
ms.date: 09/12/2018
ms.openlocfilehash: ff5da8a3d2612d7c122ec8ed87979eddf778dbf0
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902850"
---
# <a name="disaster-recovery-for-azure-applications"></a>Az Azure-alkalmazások vészhelyreállítása

A vészhelyreállítás (DR) összpontosít történő helyreállítás egy végzetes elvesztése az alkalmazás funkciói. Például ha az alkalmazást egy Azure-régió elérhetetlenné válik, szüksége egy csomag az alkalmazás fut, vagy egy másik régióban adataihoz kapcsolódó tárolási. 

Üzleti és technológiai tulajdonosok meg kell határoznia egy katasztrófa során mekkora funkció is szükséges. Ezt a szintű funkcióit is igénybe vehet néhány űrlapok: teljesen elérhetetlenné, korlátozott funkciókkal vagy késleltetett feldolgozási részben elérhető vagy teljes körűen elérhető.

Rugalmasság és a magas rendelkezésre állású stratégiák ideiglenes meghibásodás kezelésére szolgálnak.  Ez a csomag végrehajtása magában foglalja a személyeket, folyamatokat és támogató alkalmazások, amelyek lehetővé teszik a rendszer folytatja a működését. Tartalmaznia kell a csomag próba során hibák, és győződjön meg, hogy a csomag-adatbázisok a helyreállítás tesztelésére megbízható-e. 

## <a name="azure-disaster-recovery-features"></a>Azure vész-helyreállítási szolgáltatás

Az Azure biztosít a rendelkezésre állási lehetőségekért [rugalmasság műszaki útmutatást](./index.md) vész-helyreállítási támogatására. Az Azure és a vészhelyreállítás elérhetőségi funkcióit között is van egy kapcsolat. A felügyeleti szerepkörök tartalék tartomány között például egy alkalmazás rendelkezésre állásának növeli. Egy nem kezelt hardverhiba anélkül, hogy a felügyeleti "vészhelyreállítási" Ez a forgatókönyv válnak. Ezek a szolgáltatások rendelkezésre állási és stratégiák alkalmazását vészhelyreállítás nyelvi ellenőrző fontos részét képezi. Azonban ez a cikk túllép általános rendelkezésre állási problémák vészhelyreállítási események több súlyos (és ritkább).

## <a name="multiple-datacenter-regions"></a>Több adatközpont-régió
Az Azure számos régióban, a világ különböző pontjain található adatközpontokban tárolja. Ez az infrastruktúra támogatja a több vész-helyreállítási helyzetekben, például a rendszer által biztosított georeplikáció az Azure Storage másodlagos régióban. Emellett egyszerűen és alacsony költséggel valósíthatnak telepíthet egy felhőalapú szolgáltatás, a világ különböző pontjain található. Hasonlítsa össze ennek költségét és nehézségét a létrehozása és karbantartása a saját adatközpontok, több régióban a. Adatok és szolgáltatások telepítése több régióban megvédi az alkalmazás egy adott régióban jelentős meghibásodás után. A vész-helyreállítási terv kialakításakor, fontos tudni, hogy a társított két régió bármelyikén fogalmát. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure párosított régiói](/azure/best-practices-availability-paired-regions).

## <a name="azure-site-recovery"></a>Azure Site Recovery

[Az Azure Site Recovery](/azure/site-recovery/) nyújt egy egyszerű módja az Azure-beli virtuális gépek replikálása régiók között. Minimális kezelési terhelés mellett, mert nem kell a másodlagos régió minden olyan további erőforrások kiosztása rendelkezik. Ha engedélyezi a replikációt, a Site Recovery automatikusan létrehozza a szükséges erőforrásokat a célrégióban a forrás virtuális gép beállításai alapján. Automatizált folyamatos replikációt biztosít, és lehetővé teszi, hogy alkalmazás feladatátvételt egyetlen kattintással. Is futtathatja vészhelyreállítási próbák anélkül, hogy ez hatással lenne az éles számítási feladatokra vagy a folyamatban lévő replikáció, feladatátvétel tesztelésével. 

## <a name="azure-traffic-manager"></a>Azure Traffic Manager
Régióspecifikus hiba történik, amikor a forgalom kell átirányítása a szolgáltatások vagy telepítések egy másik régióban. Akkor a leghatékonyabb, kezelni a szolgáltatások, például az Azure Traffic Manager, amely automatizálja a feladatátvételt egy másik régióba érkező felhasználói forgalom, ha az elsődleges régió nem sikerül. A Traffic Manager – alapok megértése fontos egy hatékony Vészhelyreállítási stratégia tervezésekor.

A TRAFFIC Manager a tartománynévrendszer (DNS) használatával a leginkább megfelelő végpontra a forgalom-útválasztási módszer és a végpontok állapota alapján az ügyfélkéréseket. A következő ábra a felhasználók csatlakozni egy Traffic Manager URL-címet (`http://myATMURL.trafficmanager.net`) amely kivonatolja a tényleges URL-címek (`http://app1URL.cloudapp.net` és `http://app2URL.cloudapp.net`). Felhasználói kérelem irányul, a megfelelő alapul szolgáló URL-cím alapján a konfigurált [Traffic Manager útválasztási módszer](/azure/traffic-manager/traffic-manager-routing-methods). Az ebben a cikkben azt fogja érintett, csak a feladatátvételi lehetőséggel.

![Útválasztás az Azure Traffic Manager-n keresztül](./images/disaster-recovery-azure-applications/routing-using-azure-traffic-manager.png)

A Traffic Manager konfigurálásakor meg kell adnia egy új Traffic Manager DNS-előtagot, mely felhasználók fogják használni, a szolgáltatás eléréséhez. Traffic Manager most kivonatok terheléselosztás 1. szintű újabb verziója, amely a regionális szinten. A Traffic Manager DNS képez le egy olyan CNAME REKORDOT, az összes üzemelő példányhoz, amelyet felügyel.

Belül a Traffic Manager, akkor adja meg a üzemelő példánya, amely a felhasználók továbbítja a meghibásodás rangsorolt listáját. A TRAFFIC Manager figyeli a központi telepítési végpontok. Ha az elsődleges üzemelő példány nem érhető el, a Traffic Manager átirányítja a felhasználókat a következő központi telepítéshez, a prioritási listán lévő.

Bár a Traffic Manager úgy dönt, hogy hová lépjen egy feladatátvétel során, megadhatja, hogy a feladatátvétel tartomány-e alvó vagy aktív amíg Ön nem feladatátvételi módban (amely független a Traffic Manager). A TRAFFIC Manager hibát észlel, az elsődleges helyen, és áthalad a feladatátvételi helyet, függetlenül attól, hogy a hely jelenleg szolgálja ki felhasználók.

Az Azure Traffic Manager működésével kapcsolatos további információkért tekintse meg:

* [A TRAFFIC Manager áttekintése](/azure/traffic-manager/traffic-manager-overview/)
* [A Traffic Manager útválasztási módszerei](/azure/traffic-manager/traffic-manager-routing-methods)
* [Feladatátvétel útválasztási módjának konfigurálása](/azure/traffic-manager/traffic-manager-configure-failover-routing-method/)

## <a name="azure-disaster-scenarios"></a>Azure vészhelyreállítási forgatókönyvek
Az alábbi szakaszok a vészhelyreállítási forgatókönyveket számos különböző típusú terjed ki. Régióra kiterjedő szolgáltatáskimaradás nem tartoznak alkalmazásszintű hibák csak okát. Rossz kialakítás és a felügyeleti hibák valamilyen okból kimaradás lép is vezethet. Fontos figyelembe venni a hiba lehetséges okait és a tervezési és a tesztelési fázisban a helyreállítási terv során. Egy jó terv kihasználja az Azure-funkciókról, és úgy bővíti az őket az alkalmazás-specifikus stratégiák. A kiválasztott válasz határozza meg az alkalmazás fontossága a helyreállításipont-célkitűzés (RPO) és a helyreállítási időre vonatkozó célkitűzés (RTO).

### <a name="application-failure"></a>Alkalmazáshiba
Az Azure Traffic Manager automatikusan kezeli a gazdagép virtuális gép a mögöttes hardver vagy operációs rendszer szoftvert eredő hibák. Az Azure létrehoz egy új szerepkör-példányt, és hozzáadja azt a rendelkezésre álló készlet. Ha több szerepkör-példány már futott, Azure feldolgozási a többi futó szerepkörpéldányok közben, és cserélje le a sikertelen csomópontra lesz.

Súlyos hiba fordulhat elő, a hardver vagy operációs rendszer alapjául szolgáló meghibásodás nélkül. Az alkalmazás hibás logika vagy az adatok épségével kapcsolatos problémákat okozott végzetes kivétel miatt sikertelen lehet. Az alkalmazás kódjában elegendő telemetriai szerepelnie kell, hogy a monitorozási rendszer hibafeltételek észleli, és a egy alkalmazás-rendszergazda értesítést. Olyan rendszergazda, aki rendelkezik a vész-helyreállítási folyamatok teljes ismerete is döntsön feladatátvételi folyamatot indíthat, vagy fogadja el a rendelkezésre állási kimaradás a kritikus hibák feloldása közben.

### <a name="data-corruption"></a>Adatsérülés
Az Azure automatikusan tárolja az Azure SQL Database és az Azure Storage adatokat háromszor redundantly belül különböző tartalék tartományban találhatók ugyanabban a régióban. Ha georeplikációt használ, az adatok további három alkalommal tárolódik egy másik régióban. Azonban ha a felhasználók vagy az alkalmazás megsérülnek az elsődleges verziót az adatokhoz, az adatok gyorsan replikálja a többi példány. Sajnos az eredmény sérült adatok több példányát.

A lehetséges meghibásodási az adatok kezeléséhez, két lehetősége van. Először egy egyéni biztonsági mentési stratégia segítségével kezelheti. A biztonsági másolatok tárolása Azure-ban vagy a helyszínen, az üzleti követelményeit vagy szabályozási előírások függően. Egy másik lehetőség, hogy az SQL-adatbázis helyreállítása időponthoz visszaállítási lehetőség használatával. További információkért lásd: a [vész-helyreállítási stratégiát adatok](#data-strategies-for-disaster-recovery) szakaszt.

### <a name="network-outage"></a>Hálózati kimaradás
Ha az Azure-hálózat részét nem érhető el, előfordulhat, hogy nem lehet elérni az alkalmazást vagy adatokat. Ha egy vagy több szerepkörpéldányt hálózati problémák miatt nem érhető el, az Azure az alkalmazás többi rendelkezésre álló példányát használja. Ha az alkalmazás egy Azure-beli hálózati szolgáltatáskimaradás miatt nem érhető el az adatokat, potenciálisan futtathatja a csökkentett az alkalmazás funkciói helyileg gyorsítótárazott adatok használatával. Kell terveznie a vész-helyreállítási stratégiát az alkalmazás-ban korlátozott funkciókkal rendelkező futtatásához. Egyes alkalmazások esetében ez nem praktikus.

Egy másik lehetőség, hogy az adatok tárolása egy másik helyre, amíg a kapcsolat helyreáll. Ha csökkenti a funkció nem egy beállítást, a további beállításokat is alkalmazás állásidő és a egy másodlagos régióba történő feladatátvételt. Az alkalmazás korlátozott funkciókkal működik tervezési műszaki ütemezésként akár egy üzleti döntés. Ezekről a szakaszban a további [csökkenteni az alkalmazás funkciói](#reduced-application-functionality).

### <a name="failure-of-a-dependent-service"></a>Az egyik függő szolgáltatás hibája
Az Azure számos olyan szolgáltatás, amely a rendszeres szoftverkarbantartás is biztosít. Ha például [Azure Redis Cache](https://azure.microsoft.com/services/cache/) egy több-bérlős szolgáltatás, amely az alkalmazás gyorsítótárazási lehetőségeket kínál. Fontos figyelembe venni, mi történik az alkalmazásban, ha a függő szolgáltatás nem érhető el. Számos módon ebben a forgatókönyvben a hálózati szolgáltatáskimaradás forgatókönyv hasonló. Azonban az egyes szolgáltatások egymástól függetlenül eredményez az általános tervhez lehetséges fejlesztései mérlegeli.

Az Azure Redis Cache az alkalmazásnak a belül a felhőszolgáltatás üzembe helyezésének, amely katasztrófa utáni helyreállítás értékelemeket biztosít gyorsítótárazást biztosítja. Először a szolgáltatás most a szerepkörök, amelyek helyi az üzemelő példány fog futni. Ezért most már jobban tudja figyelheti és kezelheti a gyorsítótár állapotát a felügyeleti folyamatok a felhőalapú szolgáltatás részeként. Az ilyen típusú gyorsítótár is közzétesz az új funkciók, például a magas rendelkezésre állás a gyorsítótárazott adatokat, amely megőrzi a gyorsítótárban lévő adatok duplikált más csomópontokon megőrzése egyetlen csomópont meghibásodásakor.

Vegye figyelembe, hogy magas rendelkezésre állású csökkenti az átviteli sebességet, és növeli a késés, mivel az írási műveletek is kell upedate minden másodlagos példánya. A gyorsítótárazott adatok tárolásához szükséges memória hatékonyan megduplázódott, amely kell figyelembe venni kapacitásának tervezése során.  Ez a példa bemutatja, hogy minden függő szolgáltatás előfordulhat, hogy képességekkel rendelkeznek, javíthatja a rendelkezésre állás és a megrögzöttség végzetes hibák.

És minden függő szolgáltatás a szolgáltatás megszűnésének következményei tisztában kell lennie. A gyorsítótárazási példában lehet érhetők el az adatokat közvetlenül az adatbázis mindaddig, amíg a gyorsítótár visszaállítását. Ez azt eredményezi csökkentett teljesítményt, miközben alkalmazásadatok teljes hozzáférést biztosít.

### <a name="region-wide-service-disruption"></a>Régióra kiterjedő szolgáltatáskimaradás
A korábbi hibák elsősorban lett hibák, amelyek azonos Azure-régióban kezelhetők. Azonban el kell készítenie is, hogy nincs-e egy teljes régió szolgáltatáskimaradás lehetőséget. Régióra kiterjedő szolgáltatáskimaradás esetén a helyileg redundáns másolatait az adatok nem érhetők el. Engedélyezte a georeplikáció, van-e a blobok és a egy másik régióban lévő táblák három további másolatot készít. Ha a Microsoft deklarálja megszakadt a régió, Azure ismételten leképezett összes DNS-bejegyzéseket, a georeplikált régióba.

> [!NOTE]
> Vegye figyelembe, hogy nem kell minden olyan ezt a folyamatot szabályozhatja, és azt csak a régióra kiterjedő szolgáltatáskimaradás lép fel. Fontolja meg [Azure Site Recovery](/azure/site-recovery/) jobb RPO és RTO érhető el. A Site Recovery lehetővé teszi, hogy az alkalmazás döntse el, mi elfogadható kimaradás, és mikor kell a replikált virtuális gépek a feladatátvételt.

### <a name="azure-wide-service-disruption"></a>Azure kiterjedő szolgáltatáskimaradás
A vészhelyreállítás megtervezése, figyelembe kell vennie annak teljes tartományát, a lehetséges vészhelyzetek esetére. A legsúlyosabb szolgáltatáskimaradásokat közül egyszerre járna minden Azure-régióban. Csakúgy, mint más szolgáltatáskimaradásokat dönthet, hogy fogadja el ebben az esetben az ideiglenes állásidő kockázatát. Széles körű szolgáltatáskimaradásokat régiók kiterjedő olyan elkülönített szolgáltatáskimaradásokat függő szolgáltatások vagy az egyetlen régióban, mint jóval ritkább.

Azonban dönthet úgy, hogy bizonyos alapvető fontosságú alkalmazásokhoz egy többrégiós szolgáltatáskimaradás lép fel a biztonsági mentési terv szükséges. Ez a séma tartalmazhat átadni a feladatokat a szolgáltatások egy [alternatív felhőalapú](#alternative-cloud) vagy egy [a helyszíni és hibrid felhőalapú megoldással](#hybrid-on-premises-and-cloud-solution).

### <a name="reduced-application-functionality"></a>Csökkentett az alkalmazás funkciói
Egy jól megtervezett alkalmazás általában a szolgáltatások, amelyek egymással kommunikálnak, ha lazán kapcsolódó adatcsere-minták végrehajtásának használ. A DR-barát alkalmazás használatához a szolgáltatási szinten a feladatkörök elkülönítése. Ez megakadályozza, hogy az egyik függő szolgáltatás a megszakítások időtartamát a teljes alkalmazás leáll. Például vegyünk egy kereskedelmi webalkalmazás az Y vállalathoz. A következő modulok számít előfordulhat, hogy az alkalmazás:

* **Termékkatalógus** lehetővé teszi, hogy a felhasználók számára a termékek.
* **Bevásárlókocsi** lehetővé teszi a felhasználóknak a bevásárlókocsi-termékekről hozzáadása/eltávolítása.
* **ORDER Status** felhasználói rendelések szállítás állapotát jeleníti meg.
* **Beküldés ORDER** a vásárlási munkamenet finalizes fizetési megrendelés elküldésével.
* **Rendelések feldolgozása** érvényesíti a rendelés az adatok integritását, és mennyiség rendelkezésre állását ellenőrzi.

Amikor egy szolgáltatásfüggőség az alkalmazás nem érhető el, hogyan nem a szolgáltatás működőképes, amíg a függőségi állítja helyre? Egy jól megtervezett rendszer valósítja meg az elkülönítési határokat a tervezés során, és futásidőben felelősségekről, szétválasztása révén. Minden hiba elhárítása érdekében, és nem állítható helyre, rendezheti az. Helyreállíthatatlan hibák tartalomtérkép le a szolgáltatást, de csökkentheti keresztül alternatívák Helyrehozható hiba. Bizonyos problémák automatikus hibák kezeléséhez, és más műveleteket, amelyek átlátható a felhasználó. Egy komolyabb szolgáltatáskimaradás során az alkalmazás lehet teljesen nem érhető el. Egy harmadik lehetőség, hogy továbbra is a felhasználói kérelmek kezelését az korlátozott funkciókkal.

Például az adatbázist tartalmazó rendeléseket leáll, ha a rendelések feldolgozása szolgáltatás nem képes a értékesítési tranzakciók feldolgozását. Az architektúra függően nehéz vagy lehetetlen a rendelés benyújtása és a rendelések feldolgozása szolgáltatások az alkalmazás továbbra is lehet. Ha az alkalmazás nem célja e forgatókönyv, a teljes alkalmazás offline lehet, hogy lépjen. Azonban ha a termék adatai egy másik helyen található, akkor a termékkatalógus modul továbbra is használhatók termékek megtekintéséhez. Az alkalmazás egyéb részei azonban nem érhető el, például rendezés vagy készlet lekérdezéseket is.

Annak eldöntése, milyen csökkentett alkalmazás funkció érhető el egy üzleti döntés és a egy műszaki döntés. El kell döntenie, hogyan az alkalmazás ideiglenes problémákra a felhasználókat tájékoztatni fogja. A fenti példában az alkalmazás előfordulhat, hogy engedélyezi a termékek megtekintése és hozzáadni azokat egy kosár. Azonban amikor a felhasználó megpróbál e vásárolni valamit, az alkalmazás értesíti a felhasználót, hogy a rendelési funkciók átmenetileg nem érhető el. Ez ideális az ügyfél nem, de az alkalmazás kiterjedő szolgáltatáskimaradás akadályozza meg.

## <a name="data-strategies-for-disaster-recovery"></a>Vész-helyreállítási adatok stratégiák
Megfelelő adatkezeléssel vész-helyreállítási terv kihívást aspektusa. A helyreállítási folyamat során adatok helyreállítása általában a legtöbb időt vesz igénybe. Adat-helyreállítást hiba és a konzisztencia nehéz kapcsolatban felmerülő kihívások hiba után csökkenti a funkciók különböző lehetőségeit eredményez.

Egy szempont, visszaállítása vagy az alkalmazás adatokat karbantartása. Ezeket az adatokat a hivatkozást és a egy másodlagos helyen lévő tranzakciós célú fogja használni. Egy helyi központi egy költséges és hosszadalmas tervezési folyamat több régiók vészhelyreállítási stratégia megvalósításához szükséges. A legtöbb felhőszolgáltatóhoz, mint amilyen az Azure-ban, kényelmesen, könnyen engedélyezése az alkalmazások több régióban üzembe. Ezekben a régiókban vannak földrajzilag elosztott oly módon, hogy több régiók szolgáltatáskimaradás rendkívül ritka legyen-e. A stratégia az adatok régiók közötti egyike a sikeres működés érdekében minden olyan vész-helyreállítási terv befolyásoló meghatározó tényezőket.

A következő részekben bemutatjuk a katasztrófa utáni helyreállítással kapcsolatos adatok biztonsági mentése, a referenciaadatok és a tranzakciós adatok.

### <a name="backup-and-restore"></a>Biztonsági mentés és visszaállítás
Rendszeres biztonsági mentést készít az egyes vész-helyreállítási helyzetekben képes támogatni. Más tárolási erőforrások szükséges különböző módszereket.

#### <a name="sql-database"></a>SQL Database
A Basic, Standard és prémium szintű SQL Database-szinten akkor kihasználhatja időponthoz visszaállítás az adatbázis helyreállítása. További információkért lásd: [áttekintése: a felhő üzleti folytonossági és adatbázis katasztrófa utáni helyreállítás az SQL Database](/azure/sql-database/sql-database-business-continuity/). Egy másik lehetőség, az aktív Georeplikáció által használandó SQL Database-hez. Ez automatikusan replikál a adatbázis másodlagos adatbázisok ugyanazon Azure-régióban, vagy akár egy másik régióba. Ez néhány ebben a cikkben bemutatott több manuális adatok szinkronizálási módszert lehetséges alternatívát kínál. További információkért lásd: [áttekintés: SQL adatbázis aktív Georeplikációt](/azure/sql-database/sql-database-geo-replication-overview/).

Is több manuális módszer használata a biztonsági mentés és visszaállítás. Az adatbázis-Másolás parancs segítségével az adatbázis biztonsági másolatának létrehozása tranzakció-konzisztencia. Az import/export szolgáltatás támogatja a exportálása BACPAC fájlok (az adatbázisséma és a társított adatokat tartalmazó tömörített fájlok), az Azure Blob storage-ban tárolt adatbázisok Azure SQL Database is használhatja.

A beépített redundancia, az Azure Storage két replika a biztonságimásolat-fájl ugyanabban a régióban hoz létre. Azonban a biztonsági mentési folyamat futtatásának gyakoriságát határozza meg a helyreállítási Időkorlát, vagyis az adatmennyiség elveszítheti a vész. Képzeljünk el például, hogy készítsen biztonsági mentést óránként tetején, és katasztrófa-két percet, az óra elején is. 58 perc után az utolsó biztonsági másolat rögzített adatok elvesznek. Ezenkívül régióra kiterjedő szolgáltatáskimaradás ellen, másolja a BACPAC-fájl egy másodlagos régióba. Ezután lehetősége visszaállításának ezeket a biztonsági mentéseket a másodlagos régióban. További részletekért lásd: [áttekintése: a felhő üzleti folytonossági és adatbázis katasztrófa utáni helyreállítás az SQL Database](/azure/sql-database/sql-database-business-continuity/).

#### <a name="sql-data-warehouse"></a>SQL Data Warehouse
Az SQL Data Warehouse használata [geo-biztonsági mentések](/azure/sql-data-warehouse/backup-and-restore#geo-backups) vész-helyreállítási párosított régióba visszaállításához. Ezeket a biztonsági másolatokat 24 óránként megnyílik, és visszaállítási 20 percen belül a párosított régióban. Ez a funkció alapértelmezés szerint be van az összes SQL data warehouse-adattárházak számára. Az adattárház visszaállításával kapcsolatos további információkért lásd: [visszaállítása az Azure-régióból egy földrajzi PowerShell-lel](/azure/sql-data-warehouse/sql-data-warehouse-restore#restore-from-an-azure-geographical-region-using-powershell).

#### <a name="azure-storage"></a>Azure Storage
Az Azure Storage esetében egy egyéni biztonsági mentési folyamat fejlesztése, vagy használja számos külső biztonsági mentési eszközök egyikét. Vegye figyelembe, hogy a legtöbb alkalmazás tervek további hagyhatják amikor tárolási erőforrások hivatkozás egymással. Például vegyünk egy SQL-adatbázis, amely rendelkezik egy olyan oszlop, hogy az Azure Storage-blobba. A biztonsági mentések nem egyszerre következnek be, ha az adatbázis egy blobot, amely nem készült biztonsági másolat a hiba előtt mutató kell. Az alkalmazás vagy a vész-helyreállítási terv musí implementovat rozhraní folyamatok kezeléséhez a inkonzisztenciát helyreállítás után.

#### <a name="other-data-platforms"></a>Más adatplatform
Más infrastruktúra--szolgáltatásként (IaaS) üzemeltetett az adatplatformok, például a Elasticsearch vagy a MongoDB, a saját képességek és szempontok az integrált biztonsági mentési és visszaállítási folyamat létrehozásakor. Ezen adatok platformon az általános ajánlás, hogy bármilyen natív vagy elérhető integrációs-alapú replikálás vagy pillanatfelvételeinek képességeket. Ezen képességek nem létezik vagy nem megfelelő, majd vagy használata javasolt az Azure Backup szolgáltatás kombinace spravovaného a nespravovaného pillanatképeit alkalmazásadatok időponthoz másolatának létrehozásához. Minden esetben fontos határozza meg, hogyan konzisztens biztonsági másolatok eléréséhez, különösen akkor, ha az alkalmazásadatok kiterjedő több fájlrendszerek, vagy ha több meghajtó kötet kezelők vagy a szoftveres RAID használatával egyetlen fájlrendszer vannak egyesítve.

### <a name="reference-data-pattern-for-disaster-recovery"></a>Vész-helyreállítási referencia adatmintát
Referenciaadatok, amely támogatja az alkalmazás funkciói csak olvasható adatok. Ez jellemzően nem változik gyakran. Habár a biztonsági mentés és visszaállítás egy metódust régióra kiterjedő szolgáltatáskimaradás, az RTO viszonylag hosszú. Az alkalmazás egy másodlagos régióba történő telepítésekor néhány stratégiát javíthatja az RTO a referenciaadatoknál.

Mert az adatok változásának hivatkozhat ritkán, javíthatja az RTO a referenciaadatokat a másodlagos régió az állandó másolatának megtartásával. Ezzel elkerülhető, hogy egy esetleges vészhelyzet esetén a biztonsági másolatok visszaállításához szükséges időt. A több régiók vész-helyreállítási követelmények teljesítéséhez telepítenie kell az alkalmazás és a referenciaadatok együtt több régióban. Referenciaadatok szerepe magát, a külső tárhelyen, vagy olyan kombinációját is telepíthet.

Referencia üzembe helyezési adatmodelljébe számítási csomópontok implicit módon eleget tesz a katasztrófa utáni helyreállítás. Az SQL Database referenciaadat-telepítés szükséges, hogy minden egyes régióban való üzembe helyezése a referenciaadatok másolatát. Az azonos stratégia az Azure Storage vonatkozik. Telepítenie kell az elsődleges és másodlagos régiók az Azure Storage szolgáltatásban tárolt adatok másolatát.

![Adatok kiadvány referencia elsődleges és másodlagos régiók](./images/disaster-recovery-azure-applications/reference-data-publication-to-both-primary-and-secondary-regions.png)

Saját alkalmazás-specifikus biztonsági mentési rutinokat összes adatot, beleértve a referenciaadatok is meg kell valósítani. Georeplikált másolatok régióban kizárólag régióra kiterjedő szolgáltatáskimaradás használhatók. Kiterjesztett állásidő elkerülése érdekében telepítse az alkalmazásadatok alapvető fontosságú részei a másodlagos régióba. Ez a topológia egy példa: a [aktív-passzív modell](#active-passive).

### <a name="transactional-data-pattern-for-disaster-recovery"></a>Vész-helyreállítási tranzakciós adatok minta
Mód teljesen működőképes vészhelyreállítási stratégia megvalósítását, a tranzakciós adatokat a másodlagos régióba aszinkron replikáció igényel. A gyakorlati idő windows belül, amely a replikáció fordulhat elő, határozza meg az alkalmazás RPO jellemzőit. Előfordulhat, hogy ilyenkor is helyreállíthatók az adatok, amelyek a replikáció időszak alatt az elsődleges régióból történő megszakadt. Előfordulhat, hogy kell egyesíteni később a másodlagos régióba.

A következő architektúra példák néhány ötletet a feladatátviteli forgatókönyv tranzakciós adatok kezelésének alapelveit különböző módszert is biztosítanak. Fontos megjegyezni, hogy ezek a példák nem állnak teljes körű. Köztes tárolókba, például az üzenetsorok például az Azure SQL Database kell cserélni. Lehet, hogy magukat az üzenetsorokat Azure Storage vagy az Azure Service Bus-üzenetsorok (lásd: [az Azure-üzenetsorok és Service Bus-üzenetsorok - képest, és összehasonlítása](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/)). Kiszolgáló tárolási célhely is eltérő lehet, például az Azure-táblák SQL-adatbázis helyett. Feldolgozói szerepkör ezenkívül közvetítőként működik különböző lépésben lehet beszúrni. Az célja nem pontosan emulációjához architektúrák, de érdemes figyelembe venni a különböző lehetőségeket a tranzakciós adatokat és a kapcsolódó modulok helyreállítását.

#### <a name="replication-of-transactional-data-in-preparation-for-disaster-recovery"></a>Vész-helyreállítási előkészítésekor tranzakciós adatok replikálása
Fontolja meg a tranzakciós adatok tárolásához Azure Storage-üzenetsorokat használó alkalmazások. Ez lehetővé teszi a feldolgozói szerepkörök a kiszolgáló adatbázisába leválasztott architektúra a tranzakciós adatok feldolgozásához. Ehhez a tranzakciókat valamilyen ideiglenes gyorsítótárazást, ha az előtér-szerepkörök van szükség a közvetlen lekérdezés adatok. Adatvesztés tűréshatára szintjétől függően választhatja, az üzenetsorok, a database vagy az összes tárolási erőforrást replikálni. Csak adatbázis-replikáció, az az elsődleges régió leáll, ha továbbra is helyreállíthatja az adatokat a várólisták, ha az elsődleges régió fog érkezni.

Az alábbi ábrán látható az architektúra, ahol a kiszolgáló adatbázis-régiók között szinkronizálva.

![Vész-helyreállítási előkészítésekor tranzakciós adatok replikálása](./images/disaster-recovery-azure-applications/replicate-transactional-data-in-preparation-for-disaster-recovery.png)

A legnagyobb kihívás, ez az architektúra implementálása a régiók közötti replikációs stratégiát. A [Azure SQL Data Sync](/azure/sql-database/sql-database-get-started-sql-data-sync/) szolgáltatás lehetővé teszi az ilyen típusú replikáció. A jelen cikk írásakor, a szolgáltatás előzetes verzióban érhető el, és még nem javasolt éles környezetben. További információkért lásd: [áttekintése: a felhő üzleti folytonossági és adatbázis katasztrófa utáni helyreállítás az SQL Database](/azure/sql-database/sql-database-business-continuity/). Az éles környezetben egy külső megoldás beruházni vagy a saját replikációs logika létrehozása a code-ban. Függően az architektúra a replikáció lehet kétirányú, amely összetettebb.

Egy lehetséges megvalósítása processzorkihasználtsága használja, az előző példában a köztes várólista. A feldolgozói szerepkör, amely feldolgozza az adatokat a végső célhelyet, előfordulhat, hogy a módosításnak az elsődleges régióban, mind a másodlagos régióba. Ezek nem triviális feladat, és teljes körű útmutatást replikációs kód van ez a cikk nem foglalkozik. Jelentős mennyiségű időt és a tesztelés módszere a másodlagos régióba replikálja az adatokat az befektetni. További feldolgozás és tesztelési segítségével, győződjön meg arról, hogy a feladatátvételi és helyreállítási folyamatok megfelelően kezelni minden lehetséges adatinkonzisztencia vagy ismétlődő tranzakciók.

> [!NOTE]
> Ez a tanulmány a legtöbb összpontosít platform (PaaS) szolgáltatás. Hibrid alkalmazások további replikáció és a rendelkezésre állási lehetőségek azonban az Azure Virtual Machines használatához. Ezek a hibrid alkalmazások szolgáltatás (IaaS) infrastruktúra használatával futtatni az SQL Server az Azure-beli virtuális gépeken. Ez lehetővé teszi a hagyományos rendelkezésre állási megközelítések az SQL Server AlwaysOn rendelkezésre állási csoportok vagy Naplóküldést például. Egyes technikák, például az AlwaysOn, csak a helyszíni SQL Server-példányokat és az Azure-beli virtuális gépek között működik. További információkért lásd: [az SQL Server Azure virtuális gépek magas rendelkezésre állás és vészhelyreállítás helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).
>
> 

#### <a name="reduced-application-functionality-for-transaction-capture"></a>A tranzakció rögzítéshez csökkentett az alkalmazás funkciói
Fontolja meg egy második architektúrájának, korlátozott funkciókkal működik. Az alkalmazás a másodlagos régió a funkciókat, például a jelentésben, üzleti intelligenciára épülő (BI), vagy várólisták a kiürítés inaktiválja. Tranzakciós munkafolyamatok, kizárólag a legfontosabb típusú fogad a üzleti igények szerint. A rendszer rögzíti a tranzakciókat, és a várólisták írja őket. A rendszer előfordulhat, hogy halassza el az adatok feldolgozása során a szolgáltatás megszűnésének kezdeti szakaszában. Ha az elsődleges régió, a rendszer újraaktiválja a várt időtartományon belül, a feldolgozói szerepkörök az elsődleges régióban a várólistákat is kiürítési. Ez a folyamat nem kell az adatbázis egyesítését. Ha az elsődleges régió szolgáltatáskimaradás megkezdte a megengedhető ablakot, az alkalmazás megkezdheti az üzenetsorok feldolgozását.

Ebben a forgatókönyvben az adatbázist a másodlagos régióban kell egyesíteni, miután az elsődleges újraaktiválnak növekményes tranzakciós adatokat tartalmaz. Az alábbi ábrán látható, ezt a stratégiát ideiglenesen tárolja a tranzakciós adatok, az elsődleges régió visszaállításáig.

![A tranzakció rögzítéshez csökkentett az alkalmazás funkciói](./images/disaster-recovery-azure-applications/reduced-application-functionality-for-transaction-capture.png)

Adatok felügyeleti módszerek rugalmas Azure-alkalmazások további ismertetéséhez lásd: [Failsafe: Útmutató a rugalmas Felhőbeli architektúrákat](https://channel9.msdn.com/Series/FailSafe).

## <a name="deployment-topologies-for-disaster-recovery"></a>Vész-helyreállítási üzembe helyezési topológiák
Régióra kiterjedő szolgáltatáskimaradás kritikus fontosságú alkalmazások elő kell készítenie. Egy több-a régióban üzembe helyezési stratégiához funkcióját beépítse az üzemeltetés megtervezése.

Több-a régióban üzemelő példányok is járhat, IT-folyamatok az alkalmazás- és referencia adatok közzététele a másodlagos régióba egy vészhelyzetet követően. Ha az alkalmazás azonnali feladatátvételt, az üzembe helyezési folyamat egy aktív/passzív beállítás vagy egy aktív/aktív beállítást is járhat. A központi telepítési típus az alkalmazás a másodlagos régióban futó meglévő példánnyal rendelkezik. Például az Azure Traffic Manager útválasztási szolgáltatás a DNS szintjén terheléselosztási szolgáltatásokat biztosít. Képes észlelni a szolgáltatáskimaradásokat és irányíthatja a különböző régiókban, szükség esetén a felhasználókat.

Egy sikeres Azure-beli vészhelyreállításához tartalmazza, hogy a helyreállítási beépítése a kezdetektől a megoldást. A felhőbe történő helyreállítás során fellépő hibák katasztrófa, amelyek nem érhetők el a hagyományos szolgáltató további lehetőséget biztosít. Pontosabban gyorsan és dinamikusan foglalhat egy másik régióban lévő erőforrások kerülje a hiba előtt üresjárati erőforrások költségét.

A következő szakaszok különböző üzemi topológiák vész-helyreállítási terjed ki. Általában nincs egy vonatkozóan kompromisszumot jelent a nagyobb költségek vagy további rendelkezésre állási összetettsége.

### <a name="single-region-deployment"></a>Egyetlen régióban üzembe helyezés
Egy egyetlen régióban üzembe helyezési valójában nem egy vész-helyreállítási topológia, de a célja, hogy a többi architektúrák kontrasztját. Egyetlen régióban üzemelő példányok gyakoriak alkalmazásokhoz az Azure-ban; azonban azok nem felelnek meg egy vész-helyreállítási topológia követelményeinek.

Az alábbi ábra bemutatja egy Azure-régióban futó alkalmazásokhoz. Az Azure Traffic Manager és a tartalék és frissítési tartományokba alkalmazását a régión belül az alkalmazás rendelkezésre állásának növeléséhez.

![Egyetlen régióban üzembe helyezés](./images/disaster-recovery-azure-applications/single-region-deployment.png)

Ebben a forgatókönyvben az adatbázis egy meghibásodási pont. Bár az Azure belső replikákra különböző tartalék tartomány között replikálja az adatokat, a replikáció csak az ugyanazon a régión belül. Az alkalmazás elviselni nem végzetes hiba. Ha a régió elérhetetlenné válik, így hajtsa végre a tartalék tartományok, beleértve a szolgáltatás példányai és a tárolási erőforrások.

Az összes, de legalább fontos alkalmazásait megtervezi a csomag az alkalmazások üzembe helyezése több régióban. Emellett érdemes lehet az RTO és korlátozásokat, figyelembe véve, melyik üzemi topológia használandó költség.

Vessünk most vonatkozhatnak egyedi megközelítésekre tekintse meg a feladatátvétel támogatása a különböző régiók között. Ezekben a példákban az összes folyamatát írják le, két régiót használnak.

### <a name="failover-using-azure-site-recovery"></a>Feladatátvétel az Azure Site Recovery használatával

Ha engedélyezi az Azure Site Recovery használatával az Azure virtuális gép replikációja, a másodlagos régióban létrehoz több erőforrás:

- Erőforráscsoport.
- Virtuális hálózat (VNet).
- Storage-fiók. 
- Rendelkezésre állási csoportok, amely tárolja a virtuális gépek a feladatátvételt követően.

A Virtuálisgép-lemezek az elsődleges régióban lévő adatírásokat folyamatosan átkerülnek a másodlagos régióban lévő tárfiókhoz. Helyreállítási pontok létrejönnek a célként megadott tárfiók néhány perces időközönként. Kezdeményezzen feladatátvételt, ha a cél erőforrás csoport, a virtuális hálózat és a rendelkezésre állási a helyreállított virtuális gépek jönnek létre. A feladatátvétel során a bármelyik elérhető helyreállítási pontot kiválaszthatja.


### <a name="redeployment-to-a-secondary-azure-region"></a>Újbóli üzembe helyezése egy másodlagos Azure-régióba
Újbóli üzembe helyezése egy másodlagos régióba megközelítés csak az elsődleges régió alkalmazásokat és adatbázisokat futtató rendelkezik. A másodlagos régióba nincs beállítva az automatikus feladatátvételt. Így katasztrófa történik, ha meg kell üzembe helyezése a szolgáltatás az új régióban minden része. Ez magában foglalja egy felhőalapú szolgáltatás feltöltése az Azure-ba, a felhőalapú szolgáltatás telepítéséhez, az adatok visszaállítása és DNS-kiszolgálót úgy, hogy a forgalom átirányítása módosítása.

Bár ez a több-régió közül a legtöbb megfizethető, a legrosszabb RTO-jellemzőkkel rendelkezik. Ebben a modellben a szolgáltatási csomag és az adatbázis biztonsági másolatai a helyszínen vagy a másodlagos régióba az Azure Blob storage-példány. Azonban kell egy új szolgáltatás üzembe helyezése, és állítsa vissza az adatokat, mielőtt folytatja a műveletet. Még a biztonsági mentési tárolóból az adatok átvitele teljes körű automatizálását egy új adatbázis-környezet kiépítése sok időt használ fel. Adatok áthelyezése a biztonsági mentési disk storage-ból az üres adatbázist a másodlagos régióban része a legköltségesebb a visszaállítási folyamat. Akkor kell megtennie, ez azonban ahhoz, hogy az új adatbázis a egy működési állapotot, mert azt nem replikálja.

A legjobb módja, hogy a szolgáltatási csomagok tárolására a Blob storage-ban a másodlagos régióba. Ez így nem kell a az Azure-ba, mi történik, ha telepít egy helyi fejlesztői gépen, amely a csomag feltöltése. Gyorsan telepíteni a szolgáltatást egy új felhőszolgáltatást a Blob storage-ból PowerShell-parancsfájlok használatával.

Ez a beállítás akkor gyakorlati csak egy nagy RTO tűri nem kritikus fontosságú alkalmazások esetében. Például ez esetében is működik egy alkalmazást, amely lehet néhány órán keresztül van szükség, 24 órában elérhető legyen.

![Újbóli üzembe helyezése egy másodlagos Azure-régióba](./images/disaster-recovery-azure-applications/redeploy-to-a-secondary-azure-region.png)

### <a name="active-passive"></a>Aktív-passzív
Egy aktív-passzív topológia a kiválasztott számos vállalat alkalmazást. Ez a topológia a költségek viszonylag kis növekedése az RTO fejlesztései az újbóli üzembe helyezés megközelítés keresztül biztosít. Ebben a forgatókönyvben nincs újra egy elsődleges és a egy másodlagos Azure-régióban. Összes forgalmat kerül az aktív központi telepítés az elsődleges régiótól. A másodlagos régióba jobban előkészített vész-helyreállítási, mert az adatbázis működik-e a mindkét régióban. Emellett a szinkronizálási mechanizmus van közöttük. Ez a megközelítés készenléti is magában foglalhat két változata létezik: egy adatbázis csak megközelítés vagy a teljes üzembe helyezés a másodlagos régióban.

#### <a name="database-only"></a>Csak adatbázis
Az első változat az aktív-passzív topológia csak az elsődleges régióban van, egy üzembe helyezett cloud service-alkalmazás. Azonban eltérően az újbóli üzembe helyezés módszer mindkét régióban szinkronizálva legyenek az adatbázis tartalmát. (További információkért lásd a szakasz [tranzakciós adatok minta vész-helyreállítási](#transactional-data-pattern-for-disaster-recovery).) Katasztrófa történik, ha nincsenek kevesebb kapcsolatos aktiválási követelményekre vonatkoznak. Az alkalmazás elindítása a másodlagos régióban, módosítsa az új adatbázis kapcsolati karakterláncok, és módosítsa a DNS-bejegyzéseket, hogy átirányítsa a forgalom.

Az újbóli üzembe helyezés a módszert használja, mint kell már tárolt a szolgáltatáscsomagok az Azure Blob storage a másodlagos régió gyorsabb üzembe helyezéshez. Azonban nem függő díj a legtöbb, a terhet a szükséges adatbázis-visszaállítási művelet, mert az adatbázis készen áll, és fut. Ez menti a jelentős mennyiségű időt, ami egy kedvező díjszabású DR-mintát (és az egyik leggyakrabban használt).

![Csak az aktív-passzív adatbázis](./images/disaster-recovery-azure-applications/active-passive-database-only.png)

#### <a name="full-replica"></a>Teljes replika
A második változat az aktív-passzív topológia mind az elsődleges régióban, és a másodlagos régióba rendelkezik teljes körű telepítésére. A központi telepítés magában foglalja a cloud services és a egy szinkronizált adatbázis. Azonban csak az elsődleges régió aktívan kezeli a felhasználók a hálózati kérelmek. A másodlagos régióhoz csak amikor az elsődleges régióban egy szolgáltatáskimaradás aktívvá válik. Ebben az esetben minden új hálózati kérés átirányítása a másodlagos régióba. Az Azure Traffic Manager automatikusan kezelheti a feladatátvételt.

Feladatátvétel gyorsabb, mint az adatbázis csak változat történik, mert a szolgáltatás már telepítve van. Ez a topológia egy nagyon alacsony RTO biztosít. A másodlagos feladatátvételi régióban, az elsődleges régióban hiba után azonnal fel kell lennie.

Gyorsabb válaszidő, valamint az ehhez a topológiához előre kiosztja, és üzembe helyezi a biztonsági mentési szolgáltatásai, elkerülve a kevés helyet foglal le új példányokat egy katasztrófa során a lehetőségét. Ez azért fontos, ha a másodlagos Azure-régióba közelít kapacitásának határához. Nincs szolgáltatásszint-szerződés (SLA) garantálja, hogy azonnal telepíthet egy vagy több új felhőalapú szolgáltatások bármelyik régióban.

A leggyorsabb válaszidejű ebben a modellben az elsődleges és másodlagos régióban hasonló méretezési (a szerepkörpéldányok számát) kell rendelkeznie. Annak ellenére, hogy az előnnyel jár és a fel nem használt számítási példányok költséges, és ez nem feltétlenül a legtöbb körültekintő pénzügyi választás. Ez az oka jelenleg egyre gyakoribb, és a cloud services egy némileg horizontálisan le verzióját használja a másodlagos régióban. Majd gyorsan átadja a feladatokat és a másodlagos üzembe helyezés horizontális, ha szükséges. A feladatátvételi folyamat kell automatizálni, így miután az elsődleges régióban nem érhető el, további példányokat, terhelésétől függően aktiválása. Ez is járhat, például egy automatikus skálázási mechanizmus használata [a virtual machine scale sets](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

Az alábbi ábrán látható, ha az elsődleges és másodlagos régiók tartalmaz-e egy teljes mértékben üzembe helyezett felhőalapú szolgáltatás egy aktív-passzív topológiában a modellt.

![Aktív-passzív, a teljes replika](./images/disaster-recovery-azure-applications/active-passive-full-replica.png)

### <a name="active-active"></a>Aktív-aktív
Egy aktív-aktív topológia a cloud services és az adatbázis teljes mértékben telepítve mindkét régióban. Ellentétben az aktív-passzív modell mindkét régióban a felhasználói forgalom fogadására. Ez a beállítás a leggyorsabb helyreállítási idő alapján. A szolgáltatások már vannak méretezve, hogy kezelni a terhelés mellett minden egyes régió egy részét. DNS használata a másodlagos régió már engedélyezve van. Nincs további összetettséget irányíthatja a felhasználókat a megfelelő régiót módjának meghatározása. Ciklikus időszeleteléses ütemezés lehetséges. Több valószínű, hogy bizonyos felhasználók használna egy adott régióban, ahol az elsődleges verziót adataik található.

A feladatátvétel esetén egyszerűen letilthatja DNS az elsődleges régióba. Ez minden forgalmat a másodlagos régió felé irányítja.

Ez a modell még akkor is, vannak bizonyos eltérések. Ha például a következő ábra szemlélteti egy elsődleges régióban, a fő másolatot készít az adatbázisról tulajdonosa. A cloud services mindkét régióban írni az elsődleges adatbázis. A másodlagos telepítési olvashatja az elsődleges vagy a replikált adatbázisból. Ebben a példában a replikációt még csak egyirányú.

![Aktív-aktív](./images/disaster-recovery-azure-applications/active-active.png)

Egy, a fenti ábrán az aktív-aktív architektúra hátránya van. A második régiót kell hozzáférni az adatbázis az első régióban, mert a fő példány ott található. Teljesítmény jelentős mértékben csökken a régión kívül származó adatok elérésekor. A régiók közötti adatbázis-hívás érdemes valamilyen kötegelés stratégia hívások teljesítményének javítása érdekében. További információkért lásd: [kötegelés használata SQL Database-alkalmazások teljesítményének javítása érdekében](/azure/sql-database/sql-database-use-batching-to-improve-performance/).

Egy alternatív architektúra is járhat, minden régióban, közvetlenül a saját adatbázis elérése közben. Az adott modell valamilyen típusú kétirányú replikációt az egyes régiókban az adatbázis szinkronizálásához szükséges.

Az előző topológiákkal RTO általában csökkentésével növeli a költségeket és összetettséget. Az aktív-aktív topológia eltér a költség minta. Az aktív-aktív topológia esetén nem szükséges annyi példányok az elsődleges régióban, mint az aktív-passzív topológiáját. Ha az elsődleges régióban egy aktív – passzív architektúra 10 példányra van, szüksége lehet a csak egy aktív-aktív az architektúrában minden régióban 5. Mindkét régióban most már megoszthatja a terhelés. Ez lehet egy költséget takaríthat meg az aktív-passzív topológia keresztül Ha megtartja a meleg készenléti passzív régió, Várakozás a feladatátvételi 10 példányaival.

Vegye figyelembe, hogy visszaállítja az elsődleges régióban, amíg a másodlagos régióba kaphat az új felhasználók hirtelen megnő. Ha 10 000 felhasználó minden kiszolgálón amikor az elsődleges régióban egy szolgáltatáskimaradás lép fel, a másodlagos régióba hirtelen rendelkezik 20 000 felhasználóhoz kezelésére. Figyelési szabályt, a másodlagos régióban észlelnie kell a növekvő és a dupla a példányok a másodlagos régióban. Ezzel kapcsolatban további információkért lásd a szakasz [hibaészlelés](#failure-detection).

## <a name="hybrid-on-premises-and-cloud-solution"></a>A helyszíni és hibrid felhőalapú megoldással
Egy további vész-helyreállítási stratégiát az, hogy tervezhet, amelyen a helyszíni hibrid alkalmazás és a felhőben. Az alkalmazástól függően az elsődleges régió lehet bármelyik helyet. Fontolja meg az előző architektúrában, és az imagine egy helyszíni helyhez és az elsődleges vagy másodlagos régióban.

Nincsenek hibrid architektúrák áttekinthet néhány problémát. Először is ez a cikk a legtöbb foglalkozott PaaS felhőarchitektúra-sémák. Tipikus PaaS-alkalmazások az Azure-ban az Azure-ra vonatkozó szerkezeteket, például a szerepköröket, a cloud services és a Traffic Manager támaszkodnak. Az ilyen típusú PaaS-alkalmazást egy helyszíni megoldás létrehozása a jelentősen eltérő architektúra kell rendelkeznie. Ez nem feltétlenül megvalósítható, felügyeleti és költségek szempontjából.

Vész-helyreállítási hibrid megoldás azonban kevesebb kihívásokat a felhőbe, például az IaaS-alapú architektúrák áttelepített hagyományos architektúrák rendelkezik. IaaS-alkalmazások használó virtuális gépek, amelyeken közvetlen helyszíni megfelelője a felhőben. Virtuális hálózatok használatával a helyszíni hálózati erőforrásokhoz való kapcsolódás a gépek a felhőben. Ez lehetővé teszi a különböző lehetőségeket, amelyek nem lehetséges a PaaS-csak alkalmazások. Ha például az SQL Server kihasználhatja a vészhelyreállítási megoldások például az AlwaysOn rendelkezésre állási csoportok és az adatbázis-tükrözés. További információkért lásd: [magas rendelkezésre állás és vészhelyreállítás helyreállítási SQL Serverhez készült Azure-beli virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

IaaS-megoldások is biztosítanak egy egyszerűbb útvonalat a helyszíni alkalmazások számára az Azure-t a feladatátvételt. Egy teljesen működőképes alkalmazást előfordulhat, hogy rendelkezik egy meglévő helyi régióban. Azonban mi történik, ha nem rendelkezik a szükséges erőforrásokat a feladatátvételhez egy földrajzilag elkülönült régióban karbantartása? Dönthet, hogy a virtuális gépek és virtuális hálózatok használatával az Azure-ban futó alkalmazást. Ebben az esetben adja meg a folyamatokat, amelyek a felhőbeli adatok szinkronizálása. Az Azure üzemi válik a feladatátvétel végrehajtásához a másodlagos régióba. Az elsődleges régió továbbra is a helyszíni alkalmazás. IaaS-architektúrát és funkciókkal kapcsolatos további információkért lásd: a [Virtual Machines dokumentációja](https://azure.microsoft.com/documentation/services/virtual-machines/).

## <a name="alternative-cloud"></a>Alternatív felhőalapú
Vannak helyzetek, ahol a Microsoft Azure széleskörű képességeit továbbra is lehetséges, hogy nem felel meg belső megfelelőségi szabályok vagy a szervezete által megkövetelt házirendeket. Még a legjobb előkészítése és megtervezése és megvalósítása a biztonsági mentési rendszerek katasztrófa során nem megfelelőek a globális szolgáltatás megszakadását felhőszolgáltató során.

Hasonlítsa a járó bonyodalmak és költségek megnövelt rendelkezésre állást a rendelkezésre állási követelmények vonatkoznak. Egy kockázati elemzéseket végezhet, és az RTO és RPO definiálja a megoldáshoz. Ha az alkalmazás zavarják a leállás, akkor érdemes egy további felhőalapú megoldást használ. Az interneten leáll, ha egy másik felhőalapú megoldás továbbra is lehet érhető el, ha az Azure globálisan elérhetetlenné válik.

A hibrid forgatókönyv-feladatátvétel a központi telepítések a korábbi vész-helyreállítási architektúrák is egy másik felhőalapú megoldáson belül létezhet. Alternatív felhőalapú Vészhelyreállítás helyek csak olyan megoldásokat, amelyek RTO lehetővé teszi, hogy a nagyon kicsi, ha van ilyen, állásidő kell használni. Vegye figyelembe, hogy a megoldás által használt Azure-on kívülről DR-webhelyként szükséges-e a munka konfigurálása, fejlesztése, üzembe helyezése és karbantartása. Emellett akkor is nehezebb a több felhőre kiterjedő architektúra bevált eljárásokat megvalósításához. Bár a cloud platform hasonló fogalmait, az API-k és architektúrákat eltérőek.

A Vészhelyreállítás stratégiájának megbízhatóak több felhőalapú platform, hogy értékes absztrakciós réteg szerepeljenek a megoldás kialakítását. Ez így nem kell a fejlesztéséhez és karbantartásához ugyanazt az alkalmazást, másik felhőalapú platformon vészhelyzet két különböző verzióit. Mint a hibrid forgatókönyv az Azure Virtual Machines vagy az Azure Container Service használata lehet egyszerűbben ezekben az esetekben a felhőspecifikus PaaS minták használatát.

## <a name="automation"></a>Automation
Az imént említett mintáit szükséges offline telepítés gyors aktiválásának, valamint a rendszer bizonyos részeit visszaállítása. Automatizálási szkriptek erőforrások igény szerint aktiválhatja és megoldások gyors üzembe helyezését. Alábbi használja a DR-kapcsolódó automation példák [Azure PowerShell-lel](https://msdn.microsoft.com/library/azure/jj156055.aspx), de használatával a [Azure CLI-vel](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) vagy a [Service Management REST API](https://msdn.microsoft.com/library/azure/ee460799.aspx) is jó lehetőségei vannak.

Automatizálási szkriptek DR nem transzparens módon kezeli az Azure-os eleme felügyelhető. Ez a minimálisra csökkentik az emberi hibák egységes és megismételhető eredményeket hoz létre. Előre definiált Vészhelyreállítási szkripteket is egy katasztrófa során a rendszer, és azok részlegei újraépítése idő csökkentése érdekében. Nem szeretné manuálisan döntse el, a hely visszaállítása, bár le, és elveszett pénzt percenként próbál.

Tesztelje a parancsfájlokat, ismételten elejétől a végéig. Miután ellenőrizte az alapvető funkciókat, ügyeljen arra, hogy tesztelje le azokat az [vészhelyreállítási szimuláció](#disaster-simulation). Ez segít a hibák száma a parancsfájlok vagy folyamatok tárhat fel.

Az Automation szolgáltatással a legjobb, ha egy adattár, PowerShell-parancsfájlok vagy parancssori felület (CLI) Azure-beli vészhelyreállításához parancsfájlok. Egyértelműen jelölje meg, és kategorizálja őket a gyors hozzáférés céljából. Kijelöl egy elsődleges, aki kezelheti a tárházat és a parancsfájlok verziókezelését. A paraméterek és a példa a parancsfájl magyarázatok dokumentáljuk őket. Emellett győződjön meg arról, hogy Ön ezt a dokumentációt szinkronizálva legyenek az Azure-környezetek. Ez aláhúzásjeleket a célja, hogy egy elsődleges személy felelős a tárház minden része.

## <a name="failure-detection"></a>Hiba észlelése
Rendelkezésre állással és a vész-helyreállítási problémák megfelelő kezelése érdekében meg kell tudni észlelheti és diagnosztizálhatja a hibákat. Hajtsa végre a speciális kiszolgáló és a központi telepítés figyelésének gyors felismerése a rendszer vagy összetevőinek hirtelen elérhetetlenné válik. Felmérheti az általános állapotát, a felhőszolgáltatás és annak függőségeit monitoringeszközök hajthat végre a munka egy részét. Egy megfelelő Microsoft eszköz [a System Center 2016](https://www.microsoft.com/cloud-platform/system-center). Harmadik féltől származó eszközökkel is lehetővé teszi figyelési funkciókat. A legtöbb figyelési megoldások nyomon követheti a kulcsfontosságú teljesítményszámlálók és szolgáltatás rendelkezésre állása.

Bár ezek az eszközök létfontosságú, meg kell terveznie a hibák észlelése és a jelentéskészítő egy felhőszolgáltatáson belül. Megfelelően használni az Azure Diagnostics is meg kell terveznie. Egyéni teljesítményszámlálók vagy eseménynapló bejegyzések is lehet az általános stratégia részeként. Ez további adatokat gyorsan diagnosztizálni a problémát, és állítsa vissza a teljes körű képességekkel meghibásodások biztosít. A monitorozási eszközök segítségével meghatározhatja az alkalmazás állapotáról további metrikák is tartalmazza. További információkért lásd: [Azure-diagnosztika engedélyezése az Azure Cloud Servicesben](/azure/cloud-services/cloud-services-dotnet-diagnostics/). Az általános "állapotmodell" tervezéséről, lásd: [Failsafe: Útmutató a rugalmas Felhőbeli architektúrákat](https://channel9.msdn.com/Series/FailSafe).

## <a name="disaster-simulation"></a>Vészhelyreállítási szimuláció
Szimuláció tesztelés magában foglalja a kis valós beszédhelyzetek helyzetek létrehozását a munkahelyi emelet, hogy figyelje meg, miként reagálhat rájuk a csapat tagjainak. Szimuláció azt is bemutatják, hogyan hatékony megoldások vannak a helyreállítási tervben. Hajtsa végre a szimuláció úgy, hogy a létrehozott forgatókönyvek nem zavarja a tényleges üzleti közben továbbra is kedvében van például a valós helyzeteket.

Vegye figyelembe, hogy manuálisan szimulálása a rendelkezésre állási problémák az alkalmazásban a "kapcsolótábla" típusú mikroszolgáltatásokra. Például egy helyreállítható kapcsolón keresztül indítása a adatbázis-hozzáférés kivételek-sorbarendezésre modul által okozza, hogy megfelelően működni. A hálózati adapter szintjén más modulok hasonló egyszerűsített megközelítések is igénybe vehet.

A szimuláció emeli ki, amely nem megfelelően foglalkozott problémákat. A szimulált esetekben teljesen vezérelhető kell lennie. Ez azt jelenti, hogy akkor is, ha a helyreállítási terv úgy tűnik, hogy hibás, visszaállíthatja a helyzet vissza a normál anélkül, hogy ez jelentős károkat. Emellett az is fontos, hogy értesítse-e a magasabb szintű felügyeleti arról, hogy mikor és hogyan történjen a szimuláció gyakorlatok lesz végrehajtva. Ebben a csomagban kell részletesen ideje vagy erőforrása a szimuláció hatással. A mértékek a siker is meghatározhat, történő tesztelésekor a vészhelyreállítási tervet.

Ha az Azure Site Recovery használja, az Azure-ba, ellenőrizze a replikációs stratégiáját, vagy hajtson végre adatveszteség és állásidő nélkül vészhelyreállítási próbát egy feladatátvételi tesztet hajthat végre. Feladatátvételi teszt nem érinti a folyamatban lévő virtuális gépek replikációja vagy az éles környezetben.

Számos egyéb módszert is tesztelheti a vészhelyreállítási terveket. Azonban ezek többsége el egyszerűen ezen alapvető technikából változata. Ezt a célt értékelheti ki a helyreállítási terv a megvalósíthatósági vizsgálat. A részleteket az egyszerű helyreállítási tervben szereplő hézagokat felderítését a vészhelyreállítás tesztelését összpontosít.

## <a name="service-specific-guidance"></a>Szolgáltatásspecifikus útmutató

Az alábbi témakörök ismertetik a katasztrófa utáni helyreállítás adott Azure-szolgáltatások:

| Szolgáltatás | Témakör |
|---------|-------|
| Azure Database for MySQL | [Az Azure Database for MySQL üzletmenet-folytonossági funkcióinak áttekintése](/azure/mysql/concepts-business-continuity) |
| Azure Database for PostgreSQL | [Az Azure Database for PostgreSQL üzletmenet-folytonossági funkcióinak áttekintése](/azure/postgresql/concepts-business-continuity)
| Cloud Services | [Mi a teendő az Azure Cloud Servicest befolyásoló Azure szolgáltatás kiesése esetén?](/azure/cloud-services/cloud-services-disaster-recovery-guidance) |
| Cosmos DB | [Automatikus régiónkénti feladatátvétel az üzletmenet folytonossága érdekében az Azure Cosmos DB-ben](/azure/cosmos-db/regional-failover)
| Key Vault | [Az Azure Key Vault rendelkezésre állás és redundancia](/azure/key-vault/key-vault-disaster-recovery-guidance) |
|Storage | [Mi a teendő az Azure Storage leállása esetén?](/azure/storage/storage-disaster-recovery-guidance) |
| SQL Database | [Visszaállítása egy Azure SQL Database vagy feladatátvétel a másodlagos kiszolgálóra](/azure/sql-database/sql-database-disaster-recovery) |
| Virtual machines (Virtuális gépek) | [Mi a teendő abban az esetben, ha egy Azure-szolgáltatáskimaradás hatással van az Azure-beli virtuális gépek](/azure/virtual-machines/virtual-machines-disaster-recovery-guidance) |
| Virtuális hálózatok | [Virtuális hálózat – üzletmenet-folytonossági](/azure/virtual-network/virtual-network-disaster-recovery-guidance) |

