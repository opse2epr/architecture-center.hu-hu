---
title: Tervezés az önjavítást szem előtt tartva
titleSuffix: Azure Application Architecture Guide
description: A rugalmas alkalmazások manuális beavatkozás nélkül is helyre tudnak állni a hibák után.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 5e5af0be41fa892e490d556ef4286d5367144fd9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299650"
---
# <a name="design-for-self-healing"></a>Tervezés az önjavítást szem előtt tartva

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>Az alkalmazás tervezése önmaga kijavítására, ha hiba történik

Az elosztott rendszerekben időnként fellépnek hibák. Meghibásodhatnak a hardverek. Átmeneti hibák lehetnek a hálózaton. Ritkán a teljes szolgáltatásban vagy régióban üzemszünet állhat elő, de ezekre is fel kell készülni.

Ezért tervezze úgy az alkalmazását, hogy kijavítsa önmagát, ha hiba történik. Ehhez az alábbi három pillérre támaszkodó megközelítésre van szükség:

- A hibák észlelése.
- A hibákra adott megfelelő válasz.
- A hibák naplózása és monitorozása, ami információt biztosít a működéssel kapcsolatban.

Az adott hibatípusra adott válasz az alkalmazás rendelkezésreállási követelményeitől függhet. Ha például nagyon magas rendelkezésre állásra van szüksége, akkor előfordulhat, hogy automatikusan egy másodlagos régióra végez feladatátvételt egy regionális kimaradás során. Ez azonban nagyobb költséget von magával, mint az egyetlen régióban történő üzembe helyezés.

Emellett ne csak az olyan nagy eseményeket vegye figyelembe, mint a regionális kimaradások, mert ezek általában ritkán fordulnak elő. Legalább ennyire, vagy még jobban kell összpontosítania a helyi, rövid ideig tartó hibák, például a hálózati kapcsolathibák vagy a meghibásodott adatbázis-kapcsolatok kezelésére.

## <a name="recommendations"></a>Javaslatok

**A sikertelen műveletek újrapróbálása**. Az átmeneti meghibásodások bekövetkezhetnek a hálózati kapcsolat pillanatnyi megszakadásából, az adatbázis-kapcsolat megszakadásából, vagy foglalt szolgáltatás miatti időtúllépésből kifolyólag. Építsen újrapróbálkozási logikát az alkalmazásba az átmeneti hibák kezelésére. Sok Azure-szolgáltatás esetében az ügyfél SDK automatikus újrapróbálkozásokat valósít meg. További információkért lásd: [átmeneti hibák kezelése] [ transient-fault-handling] és a [újrapróbálkozási minta][retry].

**Hibás távoli szolgáltatások védelme (áramköri megszakító)**. Az átmeneti hibák után érdemes újrapróbálkozni, ha azonban a hiba továbbra is fennáll, az ahhoz vezethet, hogy túl sok hívó próbál igénybe venni egy hibás szolgáltatást. Ez egymásra épülő hibához vezethet, ahogy a kérések felhalmozódnak. Használja a [áramkör-megszakító minta] [ circuit-breaker] gyors (anélkül, hogy a távoli hívás) sikertelen művelet esetén nagy eséllyel lesz sikertelen.

**Kritikus erőforrások elkülönítése (válaszfal)**. Egy alrendszerben bekövetkező meghibásodás időnként kaszkádolás révén elterjedhet. Ilyen akkor történhet, ha egy hiba miatt bizonyos erőforrások, mint például szálak vagy szoftvercsatornák, nem szabadulnak fel időben, ami erőforrásfogyáshoz vezet. Ennek elkerülése érdekében particionálja a rendszert elkülönített csoportokra, aminek eredményeképpen az egy partícióban bekövetkező hiba nem vezet a teljes rendszer leállásához.

**Terheléskiegyenlítés végrehajtása**. Az alkalmazások adatforgalmának esetleges hirtelen kiugrásai túlterhelhetik a háttérkiszolgálón futó szolgáltatásokat. Ennek elkerülése érdekében használja a [terhelés üzenetsor-alapú terheléskiegyenlítési minta] [ load-level] az üzenetsorba a munkaelemeket aszinkron futáshoz. Az üzenetsor pufferként működik, amely csökkenti a terhelési kiugrásokat.

**Feladatátvétel**. Ha egy példány nem érhető el, végezzen feladatátvételt egy másik példányra. Az állapot nélküli elemek, például webkiszolgálók esetén, helyezzen több példányt egy terheléselosztó vagy forgalomkezelő mögé. Az állapotot tároló elemek, például adatbázisok esetén, replikákat és feladatátvételt használjon. Az adattártól és replikálási módjától függően előfordulhat, hogy az alkalmazásnak végleges konzisztenciát kell kezelnie.

**A sikertelen tranzakciók kompenzálása**. Általában kerülje az elosztott tranzakciókat, mert a szolgáltatások és erőforrások közötti koordinációt igényelnek. Helyette kisebb, önálló tranzakciókból állítsa össze a műveleteket. Ha a művelet menet közben hiúsul meg, [kompenzáló tranzakciók][compensating-transactions] használatával vonhatja vissza a már végrehajtott lépéseket.

**Ellenőrzőpontok elhelyezése a hosszú ideig futó tranzakciókban**. Az ellenőrzőpontok rugalmasságot biztosítanak, ha egy hosszú ideig futó művelet meghiúsul. Amikor a művelet újraindul (például átveszi egy másik virtuális gép), az utolsó ellenőrzőponttól folytatható.

**Leállás nélküli teljesítménycsökkenés**. Néha nem lehet megkerülni egy problémát, ilyenkor azonban biztosíthat csökkentett funkciók, amelyek továbbra is hasznosak. Vegyünk például egy könyvkatalógust megjelenítő alkalmazást. Ha az alkalmazás nem tudja lekérni a borítók miniatűrjét, helyőrző képet jeleníthet meg helyette. Előfordulhat, hogy teljes alrendszerek számítanak nem kritikus fontosságúnak az alkalmazás számára. Egy elektronikus kereskedelmi webhelyen például a termékjavaslatok megjelenítése valószínűleg kevésbé fontos, mint a megrendelések feldolgozása.

**Ügyfelek forgalmának szabályozása**. Előfordulhat, hogy néhány felhasználó túlzott terhelést okoz, ami csökkentheti az alkalmazás rendelkezésre állását más felhasználók számára. Ilyen helyzetben bizonyos ideig szabályozhatja az ügyfél forgalmát. Tekintse meg a [sávszélesség-szabályozási minta][throttle].

**Kártékony elemek blokkolása**. Egy ügyfél forgalmának szabályozása még nem jelenti azt, hogy az ügyfél rosszindulatúan viselkedett. Pusztán azt jelenti, hogy az ügyfél túllépte a szolgáltatási kvótát. Ha azonban egy ügyfél rendszeresen túllépi a kvótát, vagy egyéb helytelen magatartást tanúsít, blokkolhatja az ügyfelet. Határozzon meg egy sávon kívüli folyamatot a felhasználó számára a blokkolás feloldásának kéréséhez.

**Vezetőválasztás használata**. Amikor koordinálnia kell egy feladatot, a [vezetőválasztás][leader-election] használatával válasszon koordinátort. Így a koordinátor nem lehet kritikus hibapont. Ha a koordinátor meghiúsul, a rendszer újat választ. A vezetőválasztási algoritmus alapoktól való megvalósítása helyett vegye fontolóra egy azonnal használható megoldás, például a Zookeeper használatát.

**Tesztelés hibabeszúrással**. A sikerhez vezető utat gyakran túl sokszor tesztelik, a sikertelenséghez vezetőt azonban egyáltalán nem. Egy rendszer hosszú ideig futhat éles üzemben, mielőtt hiba történik. Hibabeszúrással tesztelheti a rendszer meghibásodásokkal szembeni rugalmasságát valódi hibák kiváltása vagy azok szimulálása révén.

**Véletlenszerű tesztelés alkalmazása**. A véletlenszerű tesztelés a hibabeszúrás fogalmát kibővítve véletlenszerűen szúr be hibákat vagy rendellenes feltételeket az éles példányokba.

Az önjavító alkalmazások készítésének strukturált megközelítését a [Rugalmas alkalmazások tervezése az Azure-hoz][resiliency-overview] című cikkben tekintheti meg.

<!-- links -->

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md
