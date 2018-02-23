---
title: "Az Azure-alkalmazások katasztrófa utáni helyreállítás"
description: "Műszaki áttekintés és alkalmazások a Microsoft Azure vész-helyreállítási megtervezésével kapcsolatos részletes információkat."
author: adamglick
ms.date: 05/26/2017
ms.openlocfilehash: 7235e752cf1b96e392a700b223d63b07c0f85b66
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="disaster-recovery-for-azure-applications"></a>Az Azure-alkalmazások katasztrófa utáni helyreállítás

Vészhelyreállítás (DR) arra irányul, végezze el az alkalmazás funkciói helyreállíthatatlan adatvesztést. Például egy Azure-régió, az alkalmazás nem érhető el, ha szüksége a tervek az alkalmazás fut, vagy egy másik régióban az adatok elérésekor. 

Üzleti és informatikai tulajdonosok meg kell határoznia, mennyi funkció is szükséges, egy olyan vészhelyzet esetén során. Ez a funkció szint is igénybe vehet néhány űrlapok: teljesen elérhetetlenné, részben elérhető csökkentett vagy késleltetett feldolgozási keresztül vagy teljes érhető el.

Rugalmasság és a magas rendelkezésre állást biztosító stratégiáikhoz célja, hogy átmeneti hiba feltételek kezelésére.  A terv végrehajtása magában foglalja a személyeket, folyamatok és támogató alkalmazások, amelyek lehetővé teszik a rendszer folytatja a működését. A tervnek mindenképp tartalmaznia kell próba során hibák, és a terv biztosításához adatbázisok helyreállítási tesztelés hang-e. 

## <a name="azure-disaster-recovery-features"></a>Azure vész-helyreállítási szolgáltatások

Mivel a rendelkezésre állási lehetőségekért Azure biztosít [rugalmassági műszaki útmutatót](./index.md) arra tervezték, hogy katasztrófa utáni helyreállítás. Kapcsolat áll fenn is az Azure és a vészhelyreállításnak rendelkezésre állása szolgáltatásai között. A tartalék tartományok közötti szerepkörök felügyeleti például egy alkalmazás rendelkezésre állásának növeli. Egy nem kezelt hardverhiba anélkül, hogy a felügyeleti válna a "vész" forgatókönyvhöz. Ezen rendelkezésre állási funkciók és stratégiák, ami fontos eleme annak a vész-ellenőrző van az alkalmazás. Azonban ez a cikk túllép általános elérhetőségével kapcsolatos problémákat katasztrófa események több súlyos (és egyes).

## <a name="multiple-datacenter-regions"></a>Több adatközpont-régiókban
Azure adatközpontjaiban a világ számos régiókban tart fenn. Ez az infrastruktúra támogatja a több vész-helyreállítási eljárással, például a rendszer által biztosított georeplikáció az Azure Storage másodlagos régióban. Is könnyen és olcsón telepíthet egy felhőalapú szolgáltatás világszerte több helyre. Hasonlítsa össze a költségek és nehezen létrehozása és karbantartása a saját adatközpontját több régióba. Adatok és a szolgáltatások telepítése több területre megvédi az alkalmazást egy fő leállás egy régió a. A vész-helyreállítási terv tervezéséhez, fontos párosított régiók fogalom megértéséhez. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure-régiókat párosítva](/azure/best-practices-availability-paired-regions).

## <a name="azure-site-recovery"></a>Azure Site Recovery

[Az Azure Site Recovery](/azure/site-recovery/) Azure virtuális gépek replikálása régiók közötti egyszerű módszert kínál. Minimumal felügyeleti terhelést, mert nem kell kiépíteni semmilyen további erőforrásokat a másodlagos régióban van. Ha engedélyezi a replikációt, Site Recovery automatikusan létrehozza a szükséges erőforrások a cél a régióban, a forrás virtuális gép beállításai alapján. Automatizált folyamatos replikálásra biztosít, és lehetővé teszi az alkalmazás feladatátvételt egyetlen kattintással. Is futtathatja katasztrófa utáni helyreállítás csukja a feladatátvétel teszteléséhez a termelési számítási feladatokhoz vagy a folyamatban lévő replikáció befolyásolása nélkül. 

## <a name="azure-traffic-manager"></a>Azure Traffic Manager
Egy régióspecifikus hiba akkor fordul elő, amikor services vagy egy másik régióban központi telepítései kell átirányítja a forgalmat. A leghatékonyabb kezelésére a szolgáltatások, például az Azure Traffic Managerben, amely automatizálja a felhasználói forgalomnak egy másik régióban a feladatátvétel az elsődleges régióban meghibásodásakor keresztül is. A Traffic Manager alapja megértése fontos egy hatékony vész-Helyreállítási stratégia tervezésekor.

A TRAFFIC Manager használ a tartománynévrendszer (DNS) ügyfél kéréseiket legmegfelelőbb végpontra irányuló forgalom-útválasztási módszert és a végpontok állapotát. Az alábbi ábra a felhasználók csatlakoznak a Traffic Manager URL-címe (`http://myATMURL.trafficmanager.net`) amely kivonatolja a tényleges URL-címek (`http://app1URL.cloudapp.net` és `http://app2URL.cloudapp.net`). Felhasználói kérelmek átirányítja a megfelelő alapul szolgáló URL-címe alapján a konfigurált [Traffic Manager útválasztási módszer](/azure/traffic-manager/traffic-manager-routing-methods). Az ebben a cikkben azt fogja érintett, csak a feladatátvételi kapcsolóval.

![Segítségével az Azure Traffic Manager útválasztási](./images/disaster-recovery-azure-applications/routing-using-azure-traffic-manager.png)

A Traffic Manager konfigurálásakor meg kell adnia egy új Traffic Manager DNS-előtagot, mely felhasználók fogják használni a szolgáltatás eléréséhez. Traffic Manager most kivonatok terheléselosztás 1. szint nagyobb, a regionális szintű. A Traffic Manager DNS egy olyan CNAME REKORDOT, amely akkor kezeli az telepítésére vonatkozó van leképezve.

Traffic Manager szoftverben, megadhatja a felhasználók továbbítja a hiba akkor fordul elő, amikor központi telepítések rangsorolt listáját. A TRAFFIC Manager figyeli a központi telepítés végpontok. Az elsődleges üzemelő példány nem érhető el, ha a Traffic Manager a következő központi telepítés a prioritási listán lévő felhasználók irányítja.

Bár a Traffic Manager úgy dönt, hogy önállóan hol találhatnak egy feladatátvétel során, eldöntheti, hogy a feladatátvétel tartomány-e aktív vagy inaktív közben nem feladatátvételi módban (amely független a Traffic Manager). A TRAFFIC Manager hibát észlel, az elsődleges helyen, és a feladatátvételi helyen, függetlenül attól, hogy, hogy a hely jelenleg szolgál felhasználók értéke.

Azure Traffic Manager működéséről további információkért tekintse meg:

* [A TRAFFIC Manager – áttekintés](/azure/traffic-manager/traffic-manager-overview/)
* [A Traffic Manager útválasztási módszerei](/azure/traffic-manager/traffic-manager-routing-methods)
* [Feladatátvétel útválasztási módjának konfigurálása](/azure/traffic-manager/traffic-manager-configure-failover-routing-method/)

## <a name="azure-disaster-scenarios"></a>Az Azure vészhelyreállítási forgatókönyvek
Az alábbi szakaszok a vészhelyreállítási forgatókönyveket számos különféle típusú foglalkozik. Régió kiterjedő szolgáltatás zavarokat nincsenek az alkalmazásszintű hibák csak okait. Gyenge tervezési és a felügyeleti hibák kimaradások esetén is vezethet. Fontos tervezési és a helyreállítási terv tesztelési fázis során fontolja meg a hiba lehetséges okait. Egy jó terv kihasználja az Azure szolgáltatásait, és az alkalmazás-specifikus stratégiák szabályozva őket az információhasználat. A kiválasztott válasz határozza meg az alkalmazás megbízhatósága egyre fontosabb szempont a helyreállítási időkorlát (RPO), és a helyreállítási idő célkitűzése (RTO).

### <a name="application-failure"></a>Alkalmazáshiba
Az Azure Traffic Manager automatikusan kezeli a gazdagép virtuális gép a mögöttes hardver vagy operációs rendszer szoftver hiba. Azure létrehoz egy új szerepkör-példányt, és hozzáadja azt a rendelkezésre álló készlet. Ha több szerepkör-példány már futott, Azure feldolgozási a többi futó szerepkörpéldányon közben a mag cseréje a hibás csomópont lesz.

Súlyos hiba akkor fordulhat elő, a hardver vagy operációs rendszer alapjául szolgáló meghibásodás nélkül. Az alkalmazás hibás programot vagy az adatok épségével kapcsolatos problémák okozta végzetes kivétel miatt sikertelen lehet. Az alkalmazás kódjának elegendő telemetriai szerepelnie kell, hogy a felügyeleti rendszer hibát észleli, és értesíti az alkalmazás-rendszergazda. Olyan rendszergazda, aki rendelkezik teljes körű ismeretekkel a vész-helyreállítási folyamatokat eldöntheti, indul el, a feladatátvételi folyamat, illetve a rendelkezésre állási kimaradás fogadni a kritikus hibák feloldása közben.

### <a name="data-corruption"></a>Adatsérülés
Azure automatikusan tárolja az Azure SQL Database és az Azure Storage adatokat háromszor redundantly belül különböző tartalék tartományok ugyanabban a régióban. A georeplikáció használja, az adatok tárolódnak a további háromszor egy másik régióban. Azonban ha a felhasználók vagy az alkalmazás megsérülnek tárolt adatokat az elsődleges verziót, az adatok gyorsan replikálja a többi példányt. Sajnos az eredmény sérült adatok többszörös lemásolását.

Kezelheti az adatok esetleges sérülését, két lehetősége van. Először egy egyéni biztonsági mentési stratégia segítségével kezelheti. A biztonsági másolatok tárolhatja az Azure vagy a helyszínen, attól függően, hogy az üzleti igényeknek és a cégirányítási szabályzat. Egy másik lehetőség egy SQL-adatbázis helyreállítása a pont időponthoz kötött visszaállítás segítségével. További információkért lásd: a [vész-helyreállítási adatok stratégiák](#data-strategies-for-disaster-recovery) az alábbi szakasz.

### <a name="network-outage"></a>Hálózati kimaradás
Nem érhetők el az Azure-hálózatot részeit, akkor is lehet nem érhető el az alkalmazás vagy az adatok. Ha hálózati problémák miatt nem érhetők el egy vagy több szerepkörpéldányt beállítani, az Azure a fennmaradó rendelkezésre álló példányok az alkalmazás használja. Ha az alkalmazás nem fér hozzá az adatok Azure hálózati kimaradás mert, potenciálisan futtathatja csökkentett alkalmazás funkciójú helyben gyorsítótárazott adatok használatával. Kell terveznie a vész-helyreállítási stratégiát az korlátozott funkciókkal az alkalmazás futtatásához. Egyes alkalmazások esetében ez nem feltétlenül gyakorlati.

Egy másik lehetőség egy adat tárolása egy másik helyre, amíg a kapcsolat visszaállítására. Funkció csökkenti lehetőség nem érhető el, a többi beállítás esetén alkalmazás állásidő és a feladatátvétel egy másodlagos régióba. A tervező az korlátozott funkciókkal futó alkalmazás mértékű üzleti döntés műszaki ütemezésként. Erről szól című szakaszban további [csökkenteni az alkalmazás funkciói](#reduced-application-functionality).

### <a name="failure-of-a-dependent-service"></a>Sikertelen volt-e a függő szolgáltatások
Azure számos olyan szolgáltatásokat biztosít, rendszeres leállásra is. Például [Azure Redis Cache](https://azure.microsoft.com/services/cache/) egy több-bérlős szolgáltatás, amely az alkalmazás gyorsítótárazási képességeket biztosít. Fontos figyelembe venni, mi történik az alkalmazásban, ha a függő szolgáltatás nem érhető el. Az sokféleképpen ebben a forgatókönyvben hasonlít a hálózati kimaradás forgatókönyv. Azonban annak eldöntéséhez, hogy minden egyes szolgáltatás egymástól függetlenül az általános tervhez lehetséges fejlesztései eredményez.

Azure Redis Cache belül a felhő szolgáltatástelepítés, amely vész-helyreállítási előnyöket biztosítja az alkalmazás a gyorsítótárazást biztosítja. A szolgáltatás először, most már fut a szerepköröket, amelyek a helyi, a központi telepítés. Ezért még jobban megfigyelése és kezelése a gyorsítótár állapotát a felügyeleti folyamatok a felhőalapú szolgáltatás részeként. Az ilyen típusú gyorsítótár is közzétesz az új funkciók, például a magas rendelkezésre állás a gyorsítótárazott adatokat, amely megőrzi a gyorsítótárazott adatokat, megőrizve a többi csomóponton duplikált egyetlen csomópont meghibásodásakor.

Vegye figyelembe, hogy a magas rendelkezésre állású csökkenti a teljesítményt, és növeli a késés, mivel az írási műveletek kell is upedate másodlagos példányokra. A gyorsítótárazott adatok tárolásához szükséges memória hatékonyan kettőzni, amely kell figyelembe kell venni a kapacitástervezés során.  Ez a példa bemutatja, hogy minden függő szolgáltatás előfordulhat, hogy olyan képességekkel rendelkeznek, javítja az általános rendelkezésre állására, és a végzetes hibák elleni védelem.

Az egyes függő szolgáltatás használata esetén meg kell ismernie a szolgáltatás szüneteltetése következményekkel járhat. A gyorsítótárazási példában esetleg az adatok közvetlenül elérje adatbázis addig, amíg a gyorsítótár visszaállítását. A csökkentett teljesítményt, ugyanakkor biztosítható a teljes körű hozzáférés az alkalmazásadatok eredményezne.

### <a name="region-wide-service-disruption"></a>A régióban kiterjedő szolgáltatáskiesés várható
A korábbi hibák elsősorban törölték az Azure-régión belül kezelhető hibák. Azonban el kell készítenie is, hogy van-e a szolgáltatás szüneteltetése a teljes régió lehetőséget. A régió kiterjedő szolgáltatás szüneteltetése akkor fordul elő, ha a helyileg redundáns másolatait az adatok nem érhetők el. Ha engedélyezett a georeplikáció, nincsenek a blobok és egy másik régióban táblák három további példányokat. Ha Microsoft deklarálja a régió elveszett, amelyek Azure összes DNS-bejegyzéseket, a georeplikált régióban.

> [!NOTE]
> Vegye figyelembe, hogy nem tudja befolyásolni bármely ezt a folyamatot, és azt csak a régió kiterjedő szolgáltatáskimaradás történik. Érdemes lehet [Azure Site Recovery](/azure/site-recovery/) jobb RPO és RTO eléréséhez. A Site Recovery lehetővé teszi, hogy az alkalmazás döntse el, hogy mi elfogadható kimaradás, és ha a replikált virtuális gépek a feladatátvételt.

### <a name="azure-wide-service-disruption"></a>Azure kiterjedő szolgáltatáskiesés várható
A vészhelyreállítás megtervezése, figyelembe kell vennie a lehetséges katasztrófák teljes skáláját. Minden Azure-régió egyidejűleg járna a legsúlyosabb károkat okozó szolgáltatás szolgáltatások közül. Akárcsak más service üzemzavarokhoz vezethet, dönthet ebben az esetben elfogadja az ideiglenes állásidő kockázatát. Széles körű szolgáltatás zavarokat régiók kiterjedő olyan sokkal ritkább mint függő szolgáltatások vagy egyetlen régiók elkülönített szolgáltatás vezetnének.

Azonban dönthet úgy, hogy bizonyos létfontosságú alkalmazások számára a több területi szolgáltatás szüneteltetése biztonsági mentési terv szükséges. A terv állhatnak szolgáltatások feladatátvétele egy [alternatív felhő](#alternative-cloud) vagy egy [a helyszíni és hibrid felhőalapú megoldás](#hybrid-on-premises-and-cloud-solution).

### <a name="reduced-application-functionality"></a>Csökkentett az alkalmazás funkciói
Egy jól megtervezett alkalmazás szolgáltatásokról, amelyek egymással kommunikálnak, ha annak a lazán összekapcsolt adatcsere végrehajtása általában használ. A vész-Helyreállítási mobilbarát az alkalmazáshoz a szolgáltatási szintű a feladatkörök elkülönítése. Ez megakadályozza, hogy a megszakítás a függő szolgáltatás a teljes alkalmazás leáll. Az Y vállalatot vegye figyelembe például kereskedelmi webalkalmazás. A következő modulok előfordulhat, hogy jelent a alkalmazás:

* **Termékkatalógus** lehetővé teszi, hogy a felhasználók számára a termékek.
* **Vásárlás bevásárlókocsiból** lehetővé teszi a felhasználóknak a saját kosár termékek hozzáadása/eltávolítása.
* **Rendelés állapot** felhasználói rendelések szállítási állapotát jeleníti meg.
* **Rendezés küldésének** finalizes a vásárlásra szolgáló munkamenet által a fizetési rendelés.
* **Feldolgozási sorrendben** érvényesíti a ahhoz, hogy az adatok integritását, és ellenőrzi a mennyiség rendelkezésre állási.

Amikor egy szolgáltatásfüggőség az alkalmazás nem érhető el, hogyan a szolgáltatás működik addig, amíg a függőség helyreállítja? Tetszetős rendszer megvalósítja az elkülönítési határokat tervezési időben, és futásidőben, a feladatkörök elkülönítése keresztül. Minden helyreállítható és helyreállíthatatlan hiba rendezheti. Helyrehozhatatlan hiba megjelenik a le a szolgáltatást, de mérséklésére vonatkozó útmutatások egy helyreállíthatatlan hiba alternatívák keresztül. Bizonyos problémák automatikus hibák kezelése és a másodlagos műveletek átlátható a felhasználó. A több súlyos szolgáltatás szüneteltetése során az alkalmazás teljesen elérhetetlenné, előfordulhat, hogy lesz. A harmadik lehetőség egy továbbra is csökkentett felhasználói kérelmek kezelését.

Például ha az adatbázis üzemeltetéséhez rendelések leáll, a feldolgozási sorrendben szolgáltatás nem képes a értékesítési tranzakciók feldolgozásához. Attól függően, hogy a architektúrára épülő nehéz vagy lehetetlen folytatja az alkalmazás a kérelem és a feldolgozási sorrendben szolgáltatások lehet. Ha az alkalmazás nem célja, hogy ez a forgatókönyv kezelni, a teljes alkalmazás előfordulhat, hogy kapcsolat nélkül. Azonban ha a termék adatok mentése más helyre, majd a termékkatalógus modul továbbra is használhatók termékek megtekintéséhez. Azonban az alkalmazás más részei nem érhetők el, például a rendezés vagy a készlet lekérdezések.

Milyen csökkentett az alkalmazás funkciói elérhető legmegfelelőbb egyben üzleti döntés és műszaki döntést hoznak. El kell döntenie, hogyan az alkalmazás tájékoztatja a felhasználókat az átmeneti problémákat. A fenti példában az alkalmazás lehetővé teheti termékek megtekintése és hozzáadását egy kosár. Azonban amikor a felhasználó megkísérli a vásárlás, az alkalmazás értesíti a felhasználót, hogy a rendezési funkció átmenetileg nem érhető el. Ez ideális, ha az ügyfél nem, de akadályozza meg az alkalmazásszintű üzemkimaradás időtartamát.

## <a name="data-strategies-for-disaster-recovery"></a>Adatok stratégiák katasztrófa utáni helyreállítás
Megfelelő adatok kezelése az egyik kihívást szempontja, hogy a vész-helyreállítási terv. A helyreállítási folyamat során adatok helyreállítása általában a legtöbb időt vesz igénybe. Funkció csökkenti a különböző választás kedvezőtlen nehéz kihívásai az adat-helyreállítás sikertelen, és konzisztencia-meghibásodás után.

Meg kell vizsgálni egy, a helyreállítási és karbantartása az alkalmazásadatok egy példányát kell. Ezeket az adatokat fogja használni a hivatkozást és a tranzakciós célból egy másodlagos helyen. Egy helyszíni telepítéséhez szükséges egy drága és hosszadalmas tervezési folyamat végrehajtásához egy több-régió vész-helyreállítási stratégiát. A legtöbb szolgáltatók, beleértve az Azure-ban kényelmesen, könnyen engedélyezése az alkalmazások központi telepítését figyelemmel több területre. Ezek a területek földrajzilag terjesztése úgy, hogy a több-régió szolgáltatáskiesés várható nagyon ritkán kell lennie. Az adatok régiók közötti kezelése stratégia egyike a sikeres működés érdekében minden vész-helyreállítási terv tényezőkről.

Az alábbi szakaszok ismertetik a vész-helyreállítással kapcsolatos adatok biztonsági mentése, a referenciaadatok és a tranzakciós adatok.

### <a name="backup-and-restore"></a>Biztonsági mentés és visszaállítás
Az alkalmazásadatok rendszeres biztonsági mentések egyes vész-helyreállítási eljárással tud támogatni. Eltérőek a tárolási erőforrások különböző eljárásokra van szükség.

#### <a name="sql-database"></a>SQL Database
Alapszintű, Standard és Premium SQL adatbázis rétegek kihasználhatja a pont időponthoz kötött visszaállítás az adatbázis helyreállításához. További információkért lásd: [áttekintése: üzleti folytonossági és az adatbázis katasztrófa utáni helyreállítás az SQL Database Felhőbeli](/azure/sql-database/sql-database-business-continuity/). Egy másik lehetőség egy SQL-adatbázis aktív Georeplikációt használja. Ez automatikusan azonos Azure-régióban vagy akár egy másik Azure-régiót másodlagos adatbázisok adatbázis-változást visszaállított replikálja. Ennek lehetséges alternatívát a néhány ebben a cikkben bemutatott több kézi adatok szinkronizálási módszert. További információkért lásd: [áttekintése: SQL adatbázis aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/).

Is több manuális módszerrel a biztonsági mentéshez használni, és állítsa vissza. Az adatbázis-másolat paranccsal tranzakciós konzisztencia az adatbázis biztonsági másolatának létrehozásához. Az import/export szolgáltatásról az Azure SQL-adatbázis, amely támogatja a exportáló adatbázisokat az Azure Blob storage szolgáltatásban tárolt BACPAC fájlok (az adatbázis-séma fájljait és kapcsolódó adatait tartalmazó tömörített fájlok) is használhatja.

A beépített redundanciát az Azure Storage két replikákat a biztonságimásolat-fájl ugyanabban a régióban hoz létre. Azonban fut, a biztonsági mentés gyakoriságát határozza meg, a helyreállítási Időkorlát, amely a vészhelyreállítási forgatókönyveket elveszhetnek adatok mennyisége. Tegyük fel, hogy készítsen biztonsági mentést óránként tetején, és a katasztrófa történik az óra elején két percet. 58 perc után utolsó biztonsági másolat rögzített adatok elvesznek. Emellett a régió kiterjedő szolgáltatás szüneteltetése elleni védelem érdekében, kell másolnia a BACPAC fájlok egy másik régióban. Ezután lehetősége nyílik a visszaállításának azokat a biztonsági másolatok a másodlagos régióban. További részletekért lásd: [áttekintése: üzleti folytonossági és az adatbázis katasztrófa utáni helyreállítás az SQL Database Felhőbeli](/azure/sql-database/sql-database-business-continuity/).

#### <a name="azure-storage"></a>Azure Storage
Az Azure Storage egy egyéni biztonsági mentési folyamat kifejlesztése, vagy használja sok külső biztonsági mentési eszközök egyikét. Ne feledje, hogy a legtöbb alkalmazás tervek további összetett szolgáltatásokkal amikor a tárolási erőforrások egymástól hivatkozás. Tegyük fel, amely rendelkezik egy olyan oszlop, hogy az Azure Storage egy blobba SQL-adatbázis. Ha a biztonsági mentések egy időben történik, az adatbázis kell blob, amely nem készült biztonsági másolat a meghibásodás előtt mutató hivatkozások. Az alkalmazás vagy a vész-helyreállítási terv folyamatok a inkonzisztenciát kezelni a helyreállítás után meg kell valósítania.

#### <a name="other-data-platforms"></a>Más adatok platformok
Egyéb infrastruktúrával,--szolgáltatás (IaaS) tárolt adatok platformok, például a Elasticsearch vagy a MongoDB, a saját képességek és szempontok az integrált biztonsági mentési és visszaállítási folyamat létrehozásakor. Ezen adatok platformok az általános ajánljuk, hogy bármely natív vagy elérhető integrációs-alapú replikálás vagy pillanatfelvételeinek képességeit. Ha ezekre a képességekre nem létezik vagy nem megfelelő, akkor érdemes Azure Backup szolgáltatás vagy a lemez felügyelt/nem felügyelt pillanatfelvételek alkalmazásadatok időpontban másolatának létrehozásához. Minden esetben fontos, különösen akkor, ha az alkalmazás adatait is több fájlok rendszer, vagy ha több meghajtó van egyesítve, a kötet kezelők vagy szoftveres RAID egyetlen fájlrendszer konzisztens biztonsági másolatok, eléréséhez, hogyan lehet.

### <a name="reference-data-pattern-for-disaster-recovery"></a>Vész-helyreállítási hivatkozás adatmintát
Referenciaadatok, amely támogatja az alkalmazás funkciói csak olvasható adatok. Ez általában nem változnak gyakran. Bár a biztonsági mentési és visszaállítási régió kiterjedő szolgáltatás zavarokat egy metódust, a RTO viszonylag hosszú. A másodlagos régióba alkalmazást telepít központilag, néhány stratégiák javítható a RTO a referenciaadatoknál.

Mivel adatváltozásokat hivatkozhat ritkán, javíthatja a RTO a referenciaadatok másodlagos régióban az állandó másolatának megtartásával. Ez megszünteti a biztonsági másolatok visszaállítása katasztrófa esetén szükséges időt. A több-régió vész-helyreállítási követelmények teljesítéséhez az alkalmazás és a referenciaadatok egyszerre több régióba kell telepítenie. A [hivatkozás adatmintát a magas rendelkezésre állású](high-availability-azure-applications.md#reference-data-pattern-for-high-availability), referenciaadatok szerepe magát, a külső tárhelyen, illetve kombinációjából is telepíthet.

A hivatkozás adatok telepítési modell belül a számítási csomópontok implicit módon a vész helyreállítási követelményének eleget tesz. Hivatkozási adatokat az SQL Database történő szükséges központi telepítését a referencia-adatok másolatát mindegyik régióhoz. Az azonos stratégia Azure Storage vonatkozik. Minden elsődleges és másodlagos régióban az Azure Storage tárolt referenciaadatok egy példányát kell telepítenie.

![Hivatkozás adatok közzétételét az elsődleges és másodlagos régiók](./images/disaster-recovery-azure-applications/reference-data-publication-to-both-primary-and-secondary-regions.png)

Meg kell valósítani a saját alkalmazás-specifikus biztonsági mentési rutinok minden adathoz, beleértve a referenciaadatoknál. Georeplikált másolatok régiók között csak egy régió kiterjedő szolgáltatáskiesés várható használnak. Hibaelhárításra megelőzése érdekében telepítenie kell az alkalmazásadatok kritikus fontosságú részei a másodlagos régióba. Ez a topológia példáért lásd: a [aktív-passzív modell](#active-passive).

### <a name="transactional-data-pattern-for-disaster-recovery"></a>Vész-helyreállítási tranzakciós adatmintát
Egy teljesen működőképes katasztrófa mód stratégia végrehajtásához a tranzakciós adatokat a másodlagos régióba aszinkron replikációját. A gyakorlati időablakokat, amelyen belül a replikáció akkor fordulhat elő, az alkalmazás RPO jellemzői határozza meg. Előfordulhat, hogy ilyenkor is helyreállíthatók az adatok a replikáció időszak alatt az elsődleges régióban megszakadt. Előfordulhat, hogy is el lehet a másodlagos régióba később egyesíteni.

A következő architektúra példák néhány ötleteket különböző módokon feladatátvétel esetén tranzakciós adatok kezelésére szolgálnak. Fontos megjegyezni, hogy ezek a példák nem teljes. Például köztes tárolókba, mint például a várólisták előfordulhat, hogy az Azure SQL Database írni. Lehet, hogy magukat az üzenetsorok Azure Storage vagy Azure Service Bus-üzenetsorok (lásd: [Azure várólisták és a Service Bus-üzenetsorok - képest és ellentétben](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/)). Kiszolgáló tárolási célhoz is változhat, például az Azure-táblákban SQL-adatbázis helyett. Feldolgozói szerepkörök emellett a különböző lépésekben közvetítőként lehet beszúrni. Nem pontosan emulálni ezek architektúrák, de kell figyelembe venni a különböző alternatívák tranzakciós adatok és a kapcsolódó modulok a helyreállítás célja.

#### <a name="replication-of-transactional-data-in-preparation-for-disaster-recovery"></a>A vész-helyreállítási előkészítése tranzakciós adatok replikálása
Vegye figyelembe a tranzakciós adatok tárolásához Azure tárüzenetsort használó alkalmazás. Ez lehetővé teszi, hogy a feldolgozói szerepkörök az adatbázishoz a leválasztott architektúrák tranzakciós adatok feldolgozására. Ehhez a tranzakciók valamilyen ideiglenes gyorsítótárazást, ha az előtér-szerepkörökhöz szükség a közvetlen lekérdezés az adatok. Attól függően, hogy az adatok-adatvesztési tűréshatár szintjét akkor célszerű használni replikálásához a várólisták, az adatbázis vagy az összes tároló-erőforrások. Az csak adatbázis-replikációval Ha az elsődleges régióban leáll, akkor továbbra is helyreállíthatja az adatokat, a várólisták ismét elérhető lesz az elsődleges régióban.

Az alábbi ábrán látható az architektúrát, ahol a kiszolgáló adatbázisában régiók szinkronizálását.

![A vész-helyreállítási előkészítése tranzakciós adatok replikálása](./images/disaster-recovery-azure-applications/replicate-transactional-data-in-preparation-for-disaster-recovery.png)

A legnagyobb kihívás az ebbe az architektúrába végrehajtási régiók közötti replikáció stratégia. A [Azure SQL adatszinkronizálás](/azure/sql-database/sql-database-get-started-sql-data-sync/) szolgáltatás lehetővé teszi, hogy a replikáció az ilyen típusú. Től írásának pillanatában Ez a szolgáltatás jelenleg előzetes verzióban érhető, és még nem javasolt éles környezetekben. További információkért lásd: [áttekintése: üzleti folytonossági és az adatbázis katasztrófa utáni helyreállítás az SQL Database Felhőbeli](/azure/sql-database/sql-database-business-continuity/). Az éles környezetben egy harmadik féltől származó megoldás beruházásának vagy a saját replikációs logika létrehozása a kódban. Attól függően, hogy az architektúra a replikáció is kétirányú, ez bonyolultabb.

Az egyik lehetséges megvalósítási tehetik használni, az előző példában a köztes várólista. A feldolgozói szerepkör, amely feldolgozza az adatokat a végső célhelyet lehet, hogy a módosítás az elsődleges régióban, mind a másodlagos régióba. Ezek nincsenek trivial feladatok, és teljes útmutatás replikációs kód nem ez a cikk a terjed. Jelentős mennyiségű időt és az adatok replikálásához a másodlagos régióba módszerrel történő tesztelése fektetnek. További feldolgozás és tesztelés révén biztos lehet abban, hogy a feladatátvételi és helyreállítási folyamatok megfelelően kezeli a lehetséges inkonzisztenciát vagy ismétlődő tranzakciók.

> [!NOTE]
> A dokumentum a legtöbb összpontosít platform (PaaS) szolgáltatás. További replikáció és a rendelkezésre állási beállításokat a hibrid alkalmazások használják, Azure virtuális gépeken. Ezen hibrid alkalmazások szolgáltatásként (IaaS) infrastruktúra használatával futtatni az SQL Server Azure virtuális gépeken. Az SQL Server AlwaysOn rendelkezésre állási csoportok vagy Naplóküldést például így hagyományos rendelkezésre állási módszerekkel. Egyes módszereket, például az AlwaysOn, csak a helyszíni SQL Server-példány és az Azure virtuális gépek között működik. További információkért lásd: [az SQL Server Azure virtuális gépek magas rendelkezésre állási és vészhelyreállítási helyreállítási](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).
> 
> 

#### <a name="reduced-application-functionality-for-transaction-capture"></a>A tranzakció rögzítéshez csökkentett az alkalmazás funkciói
Fontolja meg egy második architektúra, amely a csökkentett, működik. Az alkalmazást a másodlagos régióban inaktiválja az összes funkciót, például a jelentésekben, üzleti intelligenciával, vagy a kiürítés várólisták. Csak a tranzakciós munkafolyamatok legfontosabb típusú üzleti követelmények által definiált fogad. A rendszer a tranzakciók rögzíti, és a várólisták írja azokat. A rendszer, előfordulhat, hogy halassza el az adatok feldolgozása a szolgáltatás megszűnésének kezdeti szakaszában. Ha az elsődleges régió, a rendszer újraaktiválja a várt időn belül időszakban, a feldolgozói szerepkörök az elsődleges régióban a várólistákat is üríthetik. Ez a folyamat nem kell az adatbázis az egyesítés. Ha az elsődleges régióban szolgáltatáskiesés várható a megengedhető ablak túllép, az alkalmazás indításához a várólisták feldolgozása.

Ebben a forgatókönyvben az adatbázis másodlagos régióban úgy kell egyesíteni, miután az elsődleges aktiválódik növekményes tranzakciós adatokat tartalmaz. Az alábbi ábrán látható, ezt a stratégiát ideiglenesen tranzakciós adatok tárolására, amíg az elsődleges régióban helyreáll.

![A tranzakció rögzítéshez csökkentett az alkalmazás funkciói](./images/disaster-recovery-azure-applications/reduced-application-functionality-for-transaction-capture.png)

Rugalmas Azure-alkalmazások adatok technikák további tárgyalását lásd: [Failsafe: Útmutató a rugalmas felhő architektúrák](https://channel9.msdn.com/Series/FailSafe).

## <a name="deployment-topologies-for-disaster-recovery"></a>Vész-helyreállítási üzembe helyezési topológiák
Kritikus fontosságú alkalmazások régió kiterjedő szolgáltatás zavarokat elő kell készítenie. A régióban több központi telepítési stratégiával beépítse az üzemeltetés megtervezése.

Több-területre vonatkozó telepítések IT-folyamatok az alkalmazás- és referenciainformációkat adatok közzétételére a másodlagos régióba katasztrófa utáni vonatkozhat. Ha az alkalmazás azonnali feladatátvétel, a telepítési folyamat egy aktív/passzív telepítő vagy egy aktív telepítő vonatkozhat. Ennél a telepítési típusnál a másodlagos régióban fut az alkalmazás meglévő példánnyal rendelkezik. Például az Azure Traffic Manager útválasztási szolgáltatás a DNS-szintű terheléselosztás szolgáltatások biztosít. Azt szolgáltatás zavarokat észleli, és átirányíthatja a felhasználók különböző régiókban, amikor szükséges.

Egy sikeres Azure vész-helyreállítási tartalmazza, hogy a helyreállítási elejéről megoldásba történő létrehozása. A felhő esetén a helyreállítás során fellépő hibák egy olyan vészhelyzet esetén, amelyek nem érhetők el a hagyományos szolgáltató további lehetőségeket is kínál. Pontosabban dinamikusan és gyorsan hozzárendelhet erőforrásokat egy másik régióban található, elkerülve a meghibásodás előtt üresjárati erőforrások költsége.

A következő szakaszok különböző központi telepítési topológiák vész-helyreállítási foglalkozik. Általában nincs egy kompromisszumot nagyobb költség vagy a további rendelkezésre állás érdekében összetettségét.

### <a name="single-region-deployment"></a>Egyetlen területi központi telepítés
Egyetlen területi központi telepítés valóban nem a vész-helyreállítási topológia, de a célja, hogy ezzel szemben a más architektúrák rendelkező. Egyetlen területi központi telepítések megegyeznek az alkalmazások az Azure rendszerben; azonban ezek nem felelnek meg a vész-helyreállítási topológia követelményeinek.

A következő ábra szemlélteti az alkalmazást egy Azure-régió. Az Azure Traffic Manager és a tartalék és a frissítési tartományok használatához az alkalmazás abban a régióban rendelkezésre állásának növelése

![Egyetlen területi központi telepítés](./images/disaster-recovery-azure-applications/single-region-deployment.png)

Ebben a forgatókönyvben az adatbázis nem a hibaérzékeny pontok kialakulását. Bár az Azure replikálja a belső replikákra különböző tartalék tartományokban, a replikáció csak az ugyanabban a régióban belül. Az alkalmazás nem képes elviselni végzetes hiba. Ha a régió nem érhető el, ezért tegye a tartalék tartományok, beleértve a szolgáltatáspéldány, és a tárolási erőforrások.

Az összes legalább kritikus fontosságú alkalmazások, a megtervezi a különféle régiókban az alkalmazások telepítését tervezi. Kell RTO fontolja meg, és korlátozások annak eldöntéséhez, hogy melyik üzemi topológia használandó költsége.

Most vessen egy pillantást most egyedi megközelítésekre teszi lehetővé a feladatátvevő különböző régiókban között. Ezek a példák összes két régió segítségével folyamatát írják le.

### <a name="failover-using-azure-site-recovery"></a>Feladatátvétel az Azure Site Recovery segítségével

Ha engedélyezi az Azure Site Recovery segítségével Azure Virtuálisgép-replikációt, a másodlagos régióban létrehoz több erőforrások:

- Erőforráscsoport.
- Virtuális hálózathoz (VNet).
- Storage-fiók. 
- Rendelkezésre állási készletek tárolásához a virtuális gépek a feladatátvételt követően.

A virtuális gép lemezeken az elsődleges régióban adatírás folyamatosan átkerülnek a tárfiók másodlagos régióban. Helyreállítási pontok létrejönnek a céloldali tárfiók néhány percenként. Kezdeményezzen feladatátvételt, ha a helyreállított virtuális gépek jönnek létre a célként megadott csoportot, a virtuális hálózat és a rendelkezésre állási erőforráskészlethez. A feladatátvétel során bármely elérhető helyreállítási pont választhat.


### <a name="redeployment-to-a-secondary-azure-region"></a>Egy másodlagos Azure régióra újbóli üzembe helyezése
A módjáról lévőre egy másodlagos régióban csak az elsődleges régióban alkalmazásokat és adatbázisokat futtató rendelkezik. A másodlagos régióba nincs beállítva az automatikus feladatátvételre. Ezért ha katasztrófa történik, meg kell léptetéses fel az új szolgáltatás a kijelzők. Ez magában foglalja a felhőalapú szolgáltatás feltöltése az Azure-ba, a felhőalapú szolgáltatás telepítéséhez, az adatok helyreállításához és a forgalom átirányítása a DNS módosítása.

Bár ez a legtöbb megfizethető a több területi beállításokat, akkor a legrosszabb RTO jellemzőkkel rendelkezik. Ebben a modellben a szolgáltatás csomag és az adatbázis biztonsági másolatai a helyszínen vagy a másodlagos régióba az Azure Blob storage-példány. Azonban kell telepítenie egy új szolgáltatás, és állítsa vissza az adatokat, mielőtt folytatja a műveletet. Még a biztonsági mentési tárolóból az adatátvitel teljes körű automatizálását egy új adatbázis-környezet kiépítési igényel sok időt. Helyezze át a adatokat a biztonsági mentési lemezegység a másodlagos régióba az üres adatbázis része a legköltségesebb a visszaállítási folyamat. Kell megtennie, azonban, hogy az új adatbázis egy működési állapotot, azt nem replikálja.

A legjobb megoldás, a szolgáltatás csomagok tárolására a Blob Storage tárolóban, a másodlagos régióban. Ez szükségtelenné teszi a csomag feltöltése az Azure-ba, amely mi történik, ha telepít egy helyi fejlesztési gépről. Gyorsan telepítene a szervizcsomagok új felhőalapú szolgáltatás blobtárolóból PowerShell-parancsfájlok használatával.

Ez a beállítás akkor gyakorlati csak magas RTO tűri nem kritikus fontosságú alkalmazások esetében. Például ez is alkalmasak olyan alkalmazás, amely lehet több órán keresztül, de 24 órán belül elérhető kell megadni.

![Egy másodlagos Azure régióra újbóli üzembe helyezése](./images/disaster-recovery-azure-applications/redeploy-to-a-secondary-azure-region.png)

### <a name="active-passive"></a>Active-passive
Egy aktív-passzív topológia a választott számos vállalat alkalmazást. Ez a topológia tartalmaz javításokat az költség viszonylag kis növekedése RTO keresztül az újratelepítés módszerrel. Ilyen esetben van újra egy elsődleges és egy másodlagos Azure-régiót. Összes forgalmat állapotba aktív központi az elsődleges régióban. Mivel az adatbázis fut mindkét régió a másodlagos régióba jobban felkészül a vész-helyreállítási. Emellett a szinkronizálási mechanizmus közöttük rendelkezésre áll. Ez a megközelítés készenléti magába foglaló két változata: csak adatbázis megközelítés vagy egy másodlagos régióban teljes telepítése.

#### <a name="database-only"></a>Csak az adatbázis
Az aktív-passzív topológia első változata csak az elsődleges régióban van egy telepített felhő szolgáltatásalkalmazást. Azonban eltérően az újratelepítés módszert használja, a két régió szinkronizálva legyenek az adatbázis tartalmát. (További információkért lásd [tranzakciós adatmintát vész-helyreállítási](#transactional-data-pattern-for-disaster-recovery).) Egy olyan vészhelyzet esetén következik be, amikor nincsenek kevesebb kapcsolatos aktiválási követelményekre vonatkoznak. Az alkalmazás elindítása a másodlagos régióban, módosítsa az új adatbázis kapcsolati karakterláncok, és módosítsa a DNS-bejegyzéseket a forgalom átirányítása.

Például az újratelepítés módszerrel kell már tárolt a szervizcsomagok az Azure Blob storage gyorsabb telepítése másodlagos régióban. Azonban nem függő díj a legtöbb, a terhet a szükséges adatbázis-visszaállítási művelet, mert az adatbázis készen áll, és fut. Ezáltal jelentős mennyiségű időt, így egy megfizethető DR-mintát (és az egyik leggyakrabban használt).

![Csak az aktív-passzív adatbázis](./images/disaster-recovery-azure-applications/active-passive-database-only.png)

#### <a name="full-replica"></a>Teljes replika
A második változat aktív-passzív topológiájának mind az elsődleges régióban, és a másodlagos régióba rendelkezik teljes körű telepítésére. A központi telepítés a felhőszolgáltatások és egy szinkronizált adatbázist tartalmaz. Csak az elsődleges régióban azonban aktívan kezeli a hálózati kérelmek, a felhasználóktól. A másodlagos régióba válik aktívvá, csak akkor, ha az elsődleges régióban észlel a szolgáltatás szüneteltetése. Ebben az esetben minden új hálózati kérelmek átirányíthatja a másodlagos régióba. Az Azure Traffic Manager automatikusan kezelheti a feladatátvételt.

Feladatátvételi gyorsabb, mint az adatbázis csak variation oka, hogy a szolgáltatás már telepítve van. Ez a topológia egy nagyon alacsony RTO biztosít. A másodlagos feladatátvételi régió készen áll az elsődleges régióban hiba után azonnal kell lennie.

Gyorsabb válaszidővel, valamint ez a topológia előre foglal le, és telepíti a biztonsági mentését, elkerülve a tárhely új példányok során egy olyan vészhelyzet esetén hiánya lehetőségét. Ez azért fontos, ha a másodlagos Azure-régiót közelít kapacitásának határához. Nincs szolgáltatásszint-szerződéssel (SLA) biztosítja, hogy azonnal telepíthet egy vagy több új felhőalapú szolgáltatások bármely régióban.

A leggyorsabb válaszidejű ebben a modellben rendelkeznie kell hasonló méretezési (a szerepkörpéldányok számát) az elsődleges és másodlagos régióban. Annak ellenére, hogy a következő előnyöket a nem használt számítási példányokért fizető költséges, és ez nem feltétlenül a legtöbb probléma kivizsgálása pénzügyi választás. Emiatt gyakori felhőszolgáltatások egy kis mértékben méretezhető régebbi verziója a másodlagos régióba használatára. Ezután gyorsan feladatátvételt és a másodlagos telepítésének bővítése, ha szükséges. A feladatátvételi folyamat kell automatizálhatja, így miután az elsődleges régióban nem érhető el, további példányok, attól függően, hogy a terhelés aktiválja. Ez vonatkozhat például az automatikus skálázás mechanizmus használata [virtuálisgép-méretezési csoportok](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

Az alábbi ábrán látható, ha az elsődleges és másodlagos területeket tartalmazza-e egy teljes körűen rendszerbe állított felhőszolgáltatás egy aktív-passzív topológia a modell.

![Aktív-passzív, a teljes replika](./images/disaster-recovery-azure-applications/active-passive-full-replica.png)

### <a name="active-active"></a>Active-active
Egy aktív-aktív topológia a felhőszolgáltatás és az adatbázis teljes mértékben telepítve mindkét régióban. Az aktív-passzív modell eltérően mindkét régió felhasználói forgalom fogadására. Ez a beállítás a leggyorsabb helyreállítási adja eredményül. A szolgáltatások kezeléséhez a terhelés mellett minden egyes régió egy része már méretezése. DNS már engedélyezve van a másodlagos régióba használatára. Nincs nagyobb fokú összetettségével jár a felhasználók a megfelelő régióba irányításához módjának meghatározása. Ciklikus multiplexelés ütemezés esetleg. Valószínűbb, hogy bizonyos felhasználókra szeretné használni egy adott területre, ahol az adatokat az elsődleges példány található.

A feladatátvétel esetén csupán letiltsa DNS az elsődleges régióban. Ez továbbítja az összes forgalom a másodlagos régióba.

Még akkor is, ez a modell több változata is. Például a következő ábra szemlélteti a elsődleges régióban, amely azonos a fő másolatot az adatbázisról. A cloud services csomag mindkét régiókban írni, hogy az elsődleges adatbázis. A másodlagos központi telepítés is olvassa a elsődleges vagy replikált adatbázis. Ebben a példában a replikációt még csak egyirányú.

![Active-active](./images/disaster-recovery-azure-applications/active-active.png)

Az aktív-aktív architektúra a fenti ábrán egy hátránya van. A második régióban kell hozzáférni az adatbázis az első régióban, mert a fő példány nem található. Teljesítmény jelentős mértékben csökken egy régiót kívül származó adatok elérésekor. A kereszt-régió adatbázishívások érdemes valamilyen kötegelés stratégia az adott hívások a teljesítmény javítása érdekében. További információkért lásd: [használata a kötegelés SQL-adatbázis teljesítményének javítása érdekében](/azure/sql-database/sql-database-use-batching-to-improve-performance/).

Egy alternatív architektúra vonatkozhat minden egyes régió közvetlenül a saját adatbázis elérése közben. Valamilyen kétirányú replikációt, hogy a modellben minden régióban az adatbázis szinkronizálásához szükséges.

Az előző-topológiával rendelkező RTO általában csökkentése növeli a költségeket és a bonyolultságot. Az aktív-aktív topológia eltér a költség mintát. Az aktív-aktív topológia nem szükség lehet az elsődleges régióban annyi példány, mint az aktív-passzív topológiáját. Ha 10 elsődleges régió, egy aktív-passzív-architektúrában, szükség lehet a csak egy aktív-aktív architektúrában minden régióban 5. Mindkét régió most megoszthatja a terhelést. Ez lehet egy költséghatékony keresztül az aktív-passzív topológia Ha megtartja a meleg készenléti passzív régió, Várakozás a feladatátvételi 10 osztályt.

Vegye figyelembe, hogy csak az elsődleges régióban visszaállításához a másodlagos régióba fordulhat elő, az új felhasználók hirtelen megugrását. Ha vannak olyan 10 000 felhasználók minden kiszolgálón amikor az elsődleges régióban a szolgáltatás szüneteltetése, a másodlagos régióba hirtelen rendelkezik 20 000 felhasználót kezelésére. Állapotfigyelési szabályainak másodlagos helyének felismeri a növelése és a dupla a példányok másodlagos régióban. A további információkért lásd [tárhelyhiba-észlelés](#failure-detection).

## <a name="hybrid-on-premises-and-cloud-solution"></a>A helyszíni és hibrid felhőalapú megoldás
Egy további vész-helyreállítási stratégiát az, hogy a tervezővel egy hibrid alkalmazás, amely a helyszíni és a felhőben. Az alkalmazástól függően az elsődleges régióban lehet az egyik helyen. Fontolja meg az előző architektúrák és képzelhető el, egy helyszíni hely és az elsődleges vagy másodlagos régióban.

A hibrid architektúrák nincsenek némi kihívást. Először ebben a cikkben a legtöbb küldött PaaS architektúra kombinációját. Tipikus PaaS alkalmazások az Azure-ban Azure-specifikus megoldások, például a szerepköröket, a felhőszolgáltatások és a Traffic Manager támaszkodnak. Egy helyszíni megoldás az ilyen típusú PaaS-alkalmazás létrehozása egy jelentősen eltérő architektúra kell rendelkeznie. Ez nem lehet felügyeleti vagy költség szempontjából valósítható meg.

Vész-helyreállítási hibrid megoldás azonban a felhőbe, például az IaaS-alapú architektúrák áttelepített hagyományos architektúrák kevesebb kapcsolatban felmerülő kihívások rendelkezik. Infrastruktúra-szolgáltatási alkalmazás használja a virtuális gépek, amelyeken közvetlen helyszíni alakokat a felhőben. Virtuális hálózatok gépek való csatlakoztatásához a felhőben és a helyszíni hálózati erőforrások is használhatja. Ez lehetővé teszi a különböző lehetőségeket, amelyek nem csak a PaaS alkalmazásokat tesz lehetővé. Például az SQL Server kihasználhatja vész-helyreállítási megoldások, például az AlwaysOn rendelkezésre állási csoportok és az adatbázis-tükrözés. További információkért lásd: [magas rendelkezésre állás és katasztrófa-helyreállítás az SQL Server Azure virtuális gépekben](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

IaaS-megoldásokat is egy egyszerűbb a helyszíni alkalmazások és a feladatátvételi használható Azure az elérési útját adja meg. Lehetséges, hogy egy teljes mértékben működő alkalmazás egy meglévő helyszíni régióban. Azonban mi történik, ha nem rendelkezik az erőforrások feladatátvételhez földrajzilag különálló terület karbantartásához? Előfordulhat, hogy az alkalmazás Azure-ban futó virtuális gépek és virtuális hálózatok használata mellett dönt. Ebben az esetben adja meg a folyamatok, amelyek szinkronizálja az adatokat a felhőbe. Az Azure-telepítés majd válik a feladatátvételre használni másodlagos régióba. Az elsődleges régióban továbbra is a helyszíni alkalmazások. IaaS-architektúrát és képességeire vonatkozó további információkért lásd: a [Virtual Machines – dokumentáció](https://azure.microsoft.com/documentation/services/virtual-machines/).

## <a name="alternative-cloud"></a>Alternatív felhő
Vannak olyan helyzetek, ahol a széles körű képességeit a Microsoft Azure még nem teljesíti belső megfelelőségi szabályok vagy intézménye igényei szerint házirendeket. Még a legjobb előkészítése és biztonságimásolat-készítő rendszerek megvalósítása során egy olyan vészhelyzet esetén a tervező nem megfelelőek a felhőbeli szolgáltató a globális szolgáltatás szüneteltetése során.

Rendelkezésre állási követelmények, a költségek és mértékű rendelkezésre állást kell összehasonlítani. Hajtsa végre a kockázati elemzés, valamint határozzon meg a megoldást a RTO és a helyreállítási Időkorlát. Ha az alkalmazás zavarják a leállási, érdemes további felhőalapú megoldás. Kivéve, ha a teljes internetes leáll, egy másik felhőalapú megoldás lehet, hogy továbbra is elérhetők, ha Azure globálisan elérhetetlenné válik.

A hibrid forgatókönyvek esetében a feladatátvételt a központi telepítések a korábbi vész-helyreállítási architektúrák is létezhet belül egy másik felhőalapú megoldás. Alternatív felhő DR helyek kell használni, csak a megoldások, amelyek RTO lehetővé teszi, hogy az állásidő minimális, ha van ilyen. Vegye figyelembe, hogy a DR-webhelyként kívül Azure alkalmazó megoldás szükséges-e a további munkahelyi konfigurálása, fejleszthet, telepíteni és fenntartani. Egyúttal nehezebb a bevált gyakorlat a kereszt-architektúra valósít meg. Bár a felhőplatformokkal hasonló fogalmait, az API-k és architektúrák eltérőek.

A vész-Helyreállítási stratégia több felhőalapú platform alapul, akkor értékes ahhoz, hogy a megoldás kialakításának absztrakciós réteget szerepeljen. A szükségtelenné fejlesztésére, és ugyanahhoz az alkalmazáshoz, a másik felhőt platformok katasztrófa esetén két különböző verzióinak kezelése. A hibrid forgatókönyv esetén az Azure virtuális gépek vagy az Azure Tárolószolgáltatás használata lehet egyszerűbben ezekben az esetekben a felhő-specifikus PaaS tervek használatát.

## <a name="automation"></a>Automatizálás
A minta csak ismertettük némelyike szükséges offline központi telepítésekhez gyors aktiválásának, valamint a rendszer meghatározott részeit visszaállítása. Automatizálási parancsfájlokat az igény szerinti erőforrások aktiválhatja és gyorsan állíthat rendszerbe megoldásokat. Az automatizálási vész-Helyreállítási kapcsolatos példák használata alatt [Azure PowerShell](https://msdn.microsoft.com/library/azure/jj156055.aspx), de használatával a [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) vagy a [szolgáltatásfelügyelet REST API](https://msdn.microsoft.com/library/azure/ee460799.aspx) helyes módon is.

Automatizálási parancsfájlokat kezelheti a vész-Helyreállítási nem transzparens módon kezeli az Azure tulajdonságát. Ennek eredménye következetes és ismételhető, minimalizálja a emberi tévedések. Előre definiált vész-Helyreállítási parancsfájlokat is, hogy a rendszer és a bennük foglalt részeit során egy olyan vészhelyzet esetén idő csökkentése érdekében. Nem szeretnénk próbálja meg manuálisan mérje fel, a hely visszaállítása, amíg le, és elveszett pénz percenként.

Tesztelje a parancsfájlokat ismételten elejétől a végéig. Annak ellenőrzése után a alapvető funkciókat, ügyeljen arra, hogy tesztelje azokat [katasztrófa szimuláció](#disaster-simulation). Ez segít a parancsfájlok vagy folyamatok hibák fedik le.

Az Automation szolgáltatásban, a legjobb, ha egy PowerShell-parancsfájlok vagy parancssori felület (CLI) parancsfájlok Azure vész-helyreállítási-tárházat. Egyértelműen megjelölni, és kategorizálása őket a gyors elérés érdekében. A tárház és a parancsfájlok versioning elsődleges személyt kijelölni. Dokumentum jól igazodik ismereteket szeretnének elsajátítani a paraméterek és példákat a parancsfájlt használja őket. Bizonyosodjon meg arról, hogy Ön szinkronban tartsa az ebben a dokumentációban és az Azure-környezetekhez. Ez a célja, hogy egy elsődleges személy feladata a tárház mindegyik részének aláhúzásjeleket tartalmazhatnak.

## <a name="failure-detection"></a>Hiba észlelése
Rendelkezésre álláshoz és vészhelyreállításhoz kapcsolatos problémák kezeléséhez, észlelheti és diagnosztizálhatja a hibák képesnek kell lennie. Hajtsa végre a kiszolgáló speciális és a központi telepítés figyelésének gyors felismerése a rendszer vagy az összetevők hirtelen elérhetetlenné válik. Figyelési eszközök általános állapotát, a felhőszolgáltatás és a függőségek felmérő része a munka hajthat végre. Egy megfelelő Microsoft eszköz [System Center 2016](https://www.microsoft.com/server-cloud/products/system-center-2016/). Külső eszközöket is biztosít figyelési lehetőségek körét. A legtöbb figyelési megoldások nyomon követheti a főbb teljesítményszámlálók és a szolgáltatás rendelkezésre állása.

Annak ellenére, hogy ezek az eszközök létfontosságú, meg kell terveznie és a jelentéskészítés a felhőszolgáltatásban. Megfelelően használni az Azure Diagnostics is meg kell tervezni. Egyéni teljesítményszámlálóit vagy eseménynapló bejegyzés is lehet a teljes stratégiájának a részét. Így lehetővé teszi több adat esetén gyorsan diagnosztizálhatja a problémát, és állítsa vissza a szolgáltatás összes funkciójáról. A Hálózatfigyelő eszközök segítségével meghatározhatja az alkalmazás állapotának további metrikák is tartalmazza. További információkért lásd: [Azure Diagnostics engedélyezése az Azure Cloud Services](/azure/cloud-services/cloud-services-dotnet-diagnostics/). Olyan, átfogó "állapotmodell" tervezéséről tárgyalását lásd: [Failsafe: Útmutató a rugalmas felhő architektúrák](https://channel9.msdn.com/Series/FailSafe).

## <a name="disaster-simulation"></a>Vészhelyreállítás szimulálása
Szimuláció tesztelés magában foglalja a kis valós helyzetek létrehozását a munkahelyi emelet, és figyelje meg, hogyan reagáljon a a csoport tagjai. Szimulációja is látható, hogyan hatékony megoldások szerepelnek a helyreállítási terv. Végrehajtás szimulációja, hogy a létrehozott forgatókönyvek nem zavarja a tényleges üzleti közben továbbra is valós helyzetek például már.

Fontolja meg a "kapcsolótábla" manuálisan szimulálása elérhetőségével kapcsolatos problémákat a kérelemben egy típusú újratervezni. Például egy letölthető kapcsolón keresztül indítható el egy rendezési modul adatbázis kivételek által okozza, hogy megfelelően működni. A hálózati illesztő szinten más modulok hasonló egyszerűsített megközelítések is igénybe vehet.

A szimuláció nem megfelelően foglalkozott fellépő esetleges problémákat mutatja be. A szimulált forgatókönyvek teljesen ellenőrizhetőnek kell lenniük. Ez azt jelenti, hogy akkor is, ha a helyreállítási terv úgy tűnik, hogy hibás, visszaállíthatja a helyzet vissza a normál jelentős károsodása nélkül. Fontos továbbá, tájékoztatják a magasabb szintű felügyeleti arról, hogy mikor és hogyan a szimuláció gyakorlatok fogja végrehajtani. Ezt a tervet az idő vagy az erőforrások lesz hatással a szimuláció kell részletesen. A mértékek siker is definiálhat a vész-helyreállítási terv tesztelése során.

Ha az Azure Site Recovery használ, az Azure-ba, ellenőrizze a replikációs stratégiát, vagy a vész-helyreállítási részletezéshez adatvesztés vagy leállás nélkül végezzen feladatátvételi tesztet hajthat végre. Feladatátvételi teszt nem érinti a folyamatban lévő Virtuálisgép-replikációt vagy az éles környezetben.

Számos más módszert tesztelheti a vész-helyreállítási tervek. Azonban legtöbbjük nem egyszerűen az alábbi alapvető eljárások változata. Ez a leképezés tesztelése az, hogy a helyreállítási terv megvalósíthatósági kiértékelése. A részleteket az egyszerű helyreállítási terv hiányosságait felderítésére összpontosít vész-helyreállítási tesztelése.

## <a name="service-specific-guidance"></a>Szolgáltatással kapcsolatos útmutatást

A következő témakörök ismertetik a katasztrófa utáni helyreállítás adott Azure-szolgáltatások:

| Szolgáltatás | Témakör |
|---------|-------|
| Cloud Services | [Mi a teendő az Azure Cloud Servicest befolyásoló Azure szolgáltatás kiesése esetén?](/azure/cloud-services/cloud-services-disaster-recovery-guidance) |
| Key Vault | [Az Azure Key Vault rendelkezésre állás és redundancia](/azure/key-vault/key-vault-disaster-recovery-guidance) |
|Tárolás | [Mi a teendő, ha egy Azure Storage kimaradás során](/azure/storage/storage-disaster-recovery-guidance) |
| SQL Database | [Egy Azure SQL Database vagy feladatátvételi visszaállításához a másodlagos](/azure/sql-database/sql-database-disaster-recovery) |
| Virtual machines (Virtuális gépek) | [Mi a teendő arra az esetre, ha egy Azure szolgáltatás megszűnésének hatással van az Azure virtuális gépek](/azure/virtual-machines/virtual-machines-disaster-recovery-guidance) |
| Virtuális hálózatok | [Virtuális hálózat – az üzletmenet folytonossága](/azure/virtual-network/virtual-network-disaster-recovery-guidance) |


