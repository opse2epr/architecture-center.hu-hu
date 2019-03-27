---
title: Minimalizálja a koordinációt
titleSuffix: Azure Application Architecture Guide
description: A skálázhatóság érdekében minimalizálja a koordinációt az alkalmazásszolgáltatások között.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: e124e6720b909bafc74a4c074454ec0a3ec30c59
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299193"
---
# <a name="minimize-coordination"></a>Minimalizálja a koordinációt

## <a name="minimize-coordination-between-application-services-to-achieve-scalability"></a>A skálázhatóság érdekében minimalizálja a koordinációt az alkalmazásszolgáltatások között.

A legtöbb felhőalapú alkalmazás több alkalmazásszolgáltatásból áll, úgymint webes kezelőfelületek, adatbázisok, üzleti folyamatok, jelentés és elemzés. A skálázhatóság és megbízhatóság biztosításához mindegyik szolgáltatásnak több példányon kell futnia.

Mi történik, ha két példány néhány megosztott állapotot érintő egyidejű műveletet próbál végrehajtani? Ilyen esetekben koordináció szükséges a csomópontok között, például az ACID megvalósulásának fenntartása érdekében. Ezen az ábrán látható, hogy a `Node2` a `Node1` csomópontra vár, hogy az feloldja az adatbázis-zárolást:

![Adatbázis-zárolást diagram](./images/database-lock.svg)

A koordináció korlátozza a horizontális skálázás előnyeit, és szűk keresztmetszetet hoz létre. Ebben a példában az alkalmazás horizontális felskálázása és további példányok hozzáadása közben megnövekedett zárolási versenyt fog tapasztalni. A legrosszabb esetben az előtérbeli példányok a legtöbb idejüket a zárolásra várva fogják tölteni.

A „pontosan egyszer” szemantika a koordináció egy másik gyakori forrása. Egy megrendelést például pontosan egyszer szabad feldolgozni. Két feldolgozó új megrendelésekre vár. A `Worker1` felveszi a feldolgozási kérést. Az alkalmazásnak biztosítania kell, hogy a `Worker2` nem ismétli meg a munkát, de ha a `Worker1` összeomlik, a megrendelést nem dobja el a rendszer.

![Koordinációs diagramja](./images/coordination.svg)

A feldolgozók közötti koordinációhoz olyan mintákat is használhat, mint például a [Feladatütemező ügynök felügyeleti mintája][sas-pattern], de ebben az esetben a munka particionálása jobb módszer lehet. Minden egyes feldolgozóhoz megrendelések bizonyos tartománya van hozzárendelve (például számlázási régiónként). Ha egy feldolgozó összeomlik, egy új példány ott folytatja a feladatot, ahol az előző abbahagyta, de a példányok nem versenyeznek egymással.

## <a name="recommendations"></a>Javaslatok

**Végső konzisztencia támogatása**. Az adatok elosztásakor koordinációra van szükség az erős konzisztencia megvalósulásának kikényszerítésére. Tegyük fel például, hogy egy művelet két adatbázist frissít. Ahelyett, hogy egy tranzakciós hatókörbe helyezné, jobb, ha a rendszer képes végső konzisztenciát használni, például a [Kompenzáló tranzakció][compensating-transaction] minta használatával, a hiba utáni logikai visszavonáshoz.

**Tartományi események használata az állapot szinkronizálásához**. A [tartományi esemény][domain-event] egy olyan esemény, amely rögzíti, ha valami fontos történik a tartományban. Az érintett szolgáltatások figyelhetik az eseményt ahelyett, hogy globális tranzakciót használnának az erőforrások közötti koordinációhoz. Ennek a módszernek a használatakor a rendszernek tolerálnia kell a végső konzisztenciát (lásd az előző elemet).

**Fontolja meg a CQRS, az Események forráskezelése és hasonló minták használatát**. Ez a két minta segíthet a versengés csökkentésében az olvasási és az írási számítási feladatok között.

- A [CQRS minta][cqrs-pattern] elkülöníti az olvasási műveleteket az írási műveletektől. Néhány megvalósítás esetében az adatok olvasása fizikailag is el van különítve az adatok írásától.

- Az [Események forráskezelése mintában][event-sourcing] az állapotmódosításokat eseménysorozatként rögzíti a rendszer egy csak hozzáfűzéssel bővíthető adattárban. Egy esemény hozzáfőzése a streamhez atomi művelet, amelyhez minimális zárolásra van szükség.

Ez a két minta kiegészíti egymást. Ha a CQRS-ben található, csak írási engedéllyel rendelkező adattár az Események forráskezelése mintát használja, a csak olvasható adattár figyelheti ugyanazokat az eseményeket egy lekérdezésekhez optimalizált, az aktuális állapotot tartalmazó, olvasható pillanatkép létrehozásához. A CQRS vagy az Események forráskezelése minta alkalmazása előtt vegye figyelembe a módszer kihívást jelentő jellemzőit. További információ: [CQRS architektúrastílus][cqrs-style].

**Adatok particionálása**.  Ne helyezze minden adatát egyetlen, számos alkalmazásszolgáltatás között megosztott adatsémába. A mikroszolgáltatási architektúra úgy kényszeríti ki ezt az alapelvet, hogy minden szolgáltatást a saját adattáráért tesz felelőssé. Egy önálló adatbázison belül az adatok szilánkokba történő particionálása javíthatja az egyidejűséget, mivel az egyik adatszilánkba író szolgáltatás nem befolyásolja a másik adatszilánkba író szolgáltatást.

**Tervezzen idempotens műveleteket**. Amikor lehetséges, az alkalmazásokat úgy tervezze meg, hogy idempotensek legyenek. Így azok legalább egy szemantikával kezelhetők. Például üzenetsorba helyezheti a munkaelemeket. Ha egy feldolgozó a művelet közepén összeomlik, egy másik feldolgozó egyszerűen folytatja a munkaelemet.

**Használjon aszinkron párhuzamos feldolgozást**. Ha egy művelethez több, aszinkron módon elvégzett lépésre van szükség (például távoli szolgáltatások meghívása), lehetséges, hogy párhuzamosan hívja meg őket, és összesíti az eredményeket. Ez a módszer azt feltételezi, hogy a lépések nem függnek az előző lépés eredményétől.

**Használjon optimista egyidejűséget, ha lehetséges**. A pesszimista egyidejűség-vezérlés adatbázis-zárolást használ az ütközések megelőzése érdekében. Ez gyenge teljesítményt okozhat és csökkentheti a rendelkezésre állást. Az optimista egyidejűség-vezérléssel minden tranzakció az adat egy pillanatfelvételének másolatát módosítja. A tranzakció véglegesítése után az adatbázismotor érvényesíti a tranzakciót, és visszautasítja az adatbázis konzisztenciáját befolyásoló tranzakciókat.

Az Azure SQL Database és SQL Server [pillanatkép-elkülönítéssel][sql-snapshot-isolation] támogatja az optimista egyidejűséget. Bizonyos Azure Storage-szolgáltatások (például az [Azure Cosmos DB][cosmosdb-faq] és az [Azure Storage][storage-concurrency]) az ETagek használatán keresztül támogatják az optimista egyidejűséget.

**Fontolja meg a MapReduce vagy egyéb párhuzamos, elosztott algoritmus használatát**. Az adatoktól és az elvégzendő munka típusától függően feloszthatja a munkát független feladatokra, amelyeket párhuzamosan működő csomópontok végezhetnek el. Lásd: [Big Compute architektúrastílus][big-compute].

**Használja a vezetőválasztást a koordinációhoz**. Olyan esetben, ahol műveleteket kell koordinálnia, győződjön meg arról, hogy a koordinátor ne váljon kritikus meghibásodási ponttá az alkalmazásban. A [Vezetőválasztási minta][leader-election] használatával egy példány bármikor vezető lehet, és koordinátorként működhet. Ha a vezető összeomlik, a rendszer egy új példányt választ vezetőnek.

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