---
title: Minimalizálja a koordinációt
description: Méretezhetőség eléréséhez alkalmazásszolgáltatások összehangolását minimalizálása érdekében
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 3cab05b539612234fd8e66517b140ac5257c3e70
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/30/2018
---
# <a name="minimize-coordination"></a>Minimalizálja a koordinációt 

## <a name="minimize-coordination-between-application-services-to-achieve-scalability"></a>Méretezhetőség eléréséhez alkalmazásszolgáltatások összehangolását minimalizálása érdekében

Felhőalapú alkalmazások többségéhez áll több alkalmazásszolgáltatások &mdash; webes előtér-webkiszolgálóinak, adatbázisok, üzleti folyamatok, jelentéskészítés és elemzés, és így tovább. Eléréséhez skálázhatóságát és megbízhatóságát, ezek a szolgáltatások szabad futnia több példánya. 

Mi történik, amikor két példányt próbál egyidejű műveleteket, amelyek hatással vannak az egyes közös állapotot? Bizonyos esetekben kell koordinációs ACID garanciák megőrzése érdekében például a csomópontok között. Ezen a diagramon `Node2` arra vár, hogy `Node1` egy adatbázis-zárolás feloldására:

![](./images/database-lock.svg)

Koordinációs korlátozza a horizontális skálázhatóságot előnyeit, és létrehozza a szűk keresztmetszeteket. Ebben a példában az alkalmazás horizontális, és adja hozzá a további példányokat, látni fogja, nagyobb zárolási versenyt. A legrosszabb esetben az előtér-példányok legmagasabbak lesz a ideig vár a zárolás.

"Pontosan egyszeri" szemantikáját egy másik gyakori forrás az egyeztetés. Például egy megrendelés pontosan egyszer kell feldolgozni. Két munkavállalók új rendelések figyelő. `Worker1`szerzi be egy feldolgozási sorrendet. Az alkalmazás biztosítania kell, hogy `Worker2` nem ugyanezt a munkát, de még ha `Worker1` összeomlik, sorrendje nem dobva.

![](./images/coordination.svg)

Például használhatja mintaként [Feladatütemező ügynök felügyelő] [ sas-pattern] koordinálhatja a munkavállalók között, de ebben az esetben jobb megközelítés lehet, a munkahelyi particionálásához. Minden egyes worker bizonyos számos rendelések (például, számlázási régiónként) van hozzárendelve. Ha egy munkavégző összeomlik, egy új példány felveszi ahol abbahagyta az előző alkalmazáspéldányt, de több példánya nem vitában.

## <a name="recommendations"></a>Javaslatok

**Vezessék be a végleges konzisztencia**. Amikor adatokat, az erős konzisztencia biztosítja kényszerítéséhez koordinációs vesz igénybe. Tegyük fel például, egy művelet frissítések két adatbázis. Helyett üzembe azt egy egyetlen tranzakció hatókörében, azt jobb, ha a rendszer lehetővé teszi végleges konzisztencia, lehet, hogy a a [Compensating tranzakció] [ compensating-transaction] mintát logikailag visszavonása esetén.

**Tartomány eseményeknek a segítségével állapot szinkronizálása**. A [tartomány esemény] [ domain-event] egy eseményt, amely rögzíti, amikor esemény történik, amely jelentőséggel bír, a tartományon belül. Érintett szolgáltatások az esemény, nem pedig egy globális tranzakcióba segítségével több szolgáltatásban koordinálja a figyelheti. Ez a módszer használata esetén a rendszer a végleges konzisztencia kell működését (lásd az előző elem). 

**Vegye figyelembe például CQRS és az esemény forrás minták**. Ezek két minták között olvasási munkaterhelések versengés csökkentéséhez és a munkaterhelések írási segítségével. 

- A [CQRS mintát] [ cqrs-pattern] elkülönítésére szolgál olvasási műveleteket az írási műveletek. Az egyes megvalósítások a beolvasott adat fizikailag elkülönített az adatok írása az. 

- Az a [esemény forrás mintát][event-sourcing], állapotváltozások csak append adattárolóihoz események sorozataként fájl rögzíti. Egy esemény hozzáfűzi a stream egy atomi művelet, minimális zárolását igénylő. 

Ezek két minták egészítik ki egymást. Ha a csak írható tárban CQRS esemény forrás, a csak olvasható tároló figyelheti az azonos események olvasható pillanatképet az aktuális állapot, a lekérdezések optimalizálva. Mielőtt CQRS vagy forrás eseményt, azonban figyelembe ezt a módszert kihívásai. További információkért lásd: [CQRS architektúra stílus][cqrs-style].

**Adatok particionálása**.  Ne üzembe az összes adat, amelyet egy adatkulcsokat számos alkalmazás szolgáltatásban. Egy mikroszolgáltatások architektúra azáltal, hogy minden egyes szolgáltatás felelős a saját adattár érvénybe lépteti a elvet. Belül egy önálló adatbázis szilánkok be az adatok particionálása javíthatja a feldolgozási, mert egy shard írása szolgáltatás nincs hatással a szolgáltatás egy másik shard írásakor.

**Az idempotent műveleti terv**. Ha lehetséges, műveletek idempotent tervezése. Így azok kezelhetők: legalább egyszeri szemantika használatával. Például a munkaelemek helyezheti várólistát. Ha egy munkavégző összeomlik éppen olyan műveletet, egy másik feldolgozónak egyszerűen felveszi a munkaelemet.

**Használja az aszinkron párhuzamos feldolgozás**. Ha egy művelet aszinkron módon történik (például távoli hívások) végrehajtott több lépést igényel, valószínűleg létre tudja azokat párhuzamosan, és majd az eredmények összesítése. Ezt a módszert használja azt feltételezi, hogy az egyes lépések nem függ az előző lépés.   

**Használja a hozzáférések optimista, amikor lehetséges**. Pesszimista feldolgozási használ adatbázis zárolások ütközések elkerülése érdekében. Ez teljesítményproblémákat okozhat, és csökkentheti a rendelkezésre állási. Az egyidejű hozzáférések optimista vezérlését minden tranzakció módosítja, a másolási vagy az adatok pillanatképe. Ha a tranzakció véglegesítése, az adatbázismotor érvényesíti a tranzakció, és hatással lenne az adatbázis-konzisztencia tranzakciók elutasítja. 

Az Azure SQL Database és SQL Server támogatja az egyidejű hozzáférések optimista keresztül [pillanatkép-elkülönítés][sql-snapshot-isolation]. Bizonyos Azure storage szolgáltatások támogatja az egyidejű hozzáférések optimista révén ETag-EK, beleértve a [Azure Cosmos DB] [ cosmosdb-faq] és [Azure Storage] [ storage-concurrency].

**Távolítsa el a MapReduce vagy más párhuzamos, elosztott algoritmusok**. Attól függően, hogy az adatok és végzett munka típusú előfordulhat független feladatokat, amelyek párhuzamosan több csomópontok végzi el a munkahelyi leíró. Lásd: [nagy számítási architektúra stílus][big-compute].

**Vezető választás használata koordinációs**. Olyan esetben, ahol kell műveletek koordinálására győződjön meg arról, a koordinátor nem válik a hibaérzékeny pontok kialakulását az alkalmazásban. Használatával a [vezető választás mintát][leader-election], egy példány vezetője tetszőleges időpontban, és a koordinátor funkcionál. A kitöltés nem sikerül, ha egy új példány kijelölt vezetője kell. 
 

<!-- links -->

[big-compute]: ../architecture-styles/big-compute.md
[compensating-transaction]: ../../patterns/compensating-transaction.md
[cqrs-style]: ../architecture-styles/cqrs.md
[cqrs-pattern]: ../../patterns/cqrs.md
[cosmosdb-faq]: /azure/cosmos-db/faq
[domain-event]: https://martinfowler.com/eaaDev/DomainEvent.html
[event-sourcing]: ../../patterns/event-sourcing.md
[leader-election]: ../../patterns/leader-election.md
[sas-pattern]: ../../patterns/scheduler-agent-supervisor.md
[sql-snapshot-isolation]: /sql/t-sql/statements/set-transaction-isolation-level-transact-sql
[storage-concurrency]: https://azure.microsoft.com/blog/managing-concurrency-in-microsoft-azure-storage-2/