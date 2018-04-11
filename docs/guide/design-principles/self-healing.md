---
title: Automatikus javítás kialakítása
description: Rugalmas alkalmazások tud helyreállni hibák után beavatkozás nélkül.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 0782b65b77615f7c006724264ab0ca2d2c7c04e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-self-healing"></a>Automatikus javítás kialakítása

## <a name="design-your-application-to-be-self-healing-when-failures-occur"></a>A hibák bekövetkezésekor öngyógyító kell alkalmazás megtervezése

Elosztott rendszer esetén fordulhat elő hibák. Hardver sikertelen lehet. A hálózati is átmeneti hibák fordultak elő. Ritkán az egész szolgáltatás vagy a régió problémákat tapasztalhat a szüneteltetése, de még az kell lennie a tervezett.

Ezért tervezzen egy alkalmazás hibák bekövetkezésekor öngyógyító. Ehhez szükséges a három pronged megközelítés:

- Észleli a hibáit.
- Hibák szabályosan válaszolni.
- Naplózhatja és megfigyelheti sikertelen, működési feltárhatja.

Hiba egy bizonyos típusú adott válasz függ, hogy az alkalmazás rendelkezésre állási követelményeinek. Például ha magas rendelkezésre állást igényel, akkor előfordulhat, hogy automatikusan áthelyezze a feladatokat egy másodlagos régióban regionális kimaradás során. Azonban, hogy egy költségesebb, mint egyetlen területi központi telepítések gyakorisága. 

Nem csupán figyelembe venni nagy eseményekkel – például regionális kimaradások esetén, amelyek általában ritkán fordul elő. Irányul, lehető legtöbb, ha nem több, a helyi, rövid élettartamú hibák, például a hálózati csatlakozási hibák kezelése, vagy nem sikerült a Helyadatbázis-kapcsolatot.

## <a name="recommendations"></a>Javaslatok

**Próbálja meg újra a sikertelen műveleteket**. Amikor egy szolgáltatás foglalt átmeneti hibák pillanatnyi megszűnését hálózati kapcsolatot, az eldobott adatbázis-kapcsolat vagy időtúllépés miatt fordulhat elő. Build újrapróbálkozási logika átmeneti hibák kezeléséhez az alkalmazásba. Az Azure-szolgáltatások számos az ügyfél SDK automatikus újrapróbálkozások valósítja meg. További információkért lásd: [átmeneti hiba kezelési] [ transient-fault-handling] és [újra mintát][retry].

**Hibás távoli szolgáltatások (áramköri megszakító) védelme**. Hasznos lehet, hogy átmeneti hiba után próbálkozzon újra, de ha a hiba továbbra is fennáll, akkor is végül beütés hibás szolgáltatás túl sok hívókat. Ez vezethet kaszkádolt sikertelen kérelmek, készítsen biztonsági másolatot. Használja a [áramköri megszakító mintát] [ circuit-breaker] gyors (nélkül a távoli hívás) meghiúsul egy művelet esetén valószínűleg hiba fog előfordulni.  

**Különítse el a fontos erőforrásokhoz (válaszfal)**. Egy alrendszer hibáinak néha kaszkádolt is. Ez akkor fordulhat elő, ha a hiba miatt egyes erőforrásokat, például a szálak vagy sockets, nem az időben, ami Erőforrásfogyás felszabadított beolvasása. Ennek elkerülése érdekében partíció egy rendszer elkülönített csoportokba, így egy partíció meghibásodása esetén nem áll le a teljes rendszer.  

**Hajtsa végre a terhelés simítás**. Alkalmazások a forgalmat, amely képes ne terhelje tovább a háttérkiszolgálón szolgáltatások hirtelen teljesítményt tapasztalhat. Ennek elkerülése érdekében használja a [várólista alapú terhelés simítás mintát] [ load-level] a várólistába helyezni a munkaelemet az aszinkron módon futnak. A várólista kisimítja a terhelés csúcsait kimenő puffer funkcionál. 

**Feladatok átadása a**. Példánya nem érhető el, ha átadja egy másik példánya. Az állapot nélküli alkalmazások, például a webkiszolgáló fog helyezze el a traffic manager vagy terheléselosztó mögött több példányt. Dolgot tároló állapota, például egy adatbázis-replikákat használ, valamint a feladatátvétel. Attól függően, hogy a tárolót, és hogyan replikált elképzelhető, hogy szükség az alkalmazás a végleges konzisztencia kezelésére. 

**Nem sikerült tranzakciók kártalanítja**. Az elosztott tranzakciók általában elkerülése, szolgáltatások és erőforrások koordinációs van szükségük. Ehelyett állítható össze a kisebb az egyes tranzakciók művelet. Ha a művelet sikertelen félúton keresztül, [Compensating tranzakciók] [ compensating-transactions] visszavonása bármely lépése, amely már befejeződött. 

**Ellenőrzőpont hosszú ideje tartó tranzakciók**. Az ellenőrzőpontok is biztosít rugalmasságot, ha egy hosszú ideig futó művelet sikertelen lesz. Ha a művelet újraindítja (például, hogy van észlelnie másik virtuális gép), a legutóbbi ellenőrzőponttól folytatni.

**Csökkentheti a szabályosan**. Néha nem dolgozunk a probléma, de megadhat csökkentett továbbra is hasznos. Érdemes lehet olyan alkalmazás, amely a katalógus könyvek mutatja. Ha az alkalmazás nem olvasható be a lefedi a miniatűr képére, azt egy helyőrző kép lehet, hogy megjelenítése. Előfordulhat, hogy az alkalmazás nem kritikus teljes alrendszereket. Például az elektronikus kereskedelmi webhely termék javaslatok megjelenítése fontos valószínűleg kevesebb mint feldolgozási rendeléseket.

**Ügyfelek szabályozás**. A felhasználók kevés néha nagy a terhelés, ami csökkentheti az alkalmazás rendelkezésre állásának más felhasználók létrehozása. Ebben a helyzetben szabályozás az ügyfél egy bizonyos ideig. Lásd: [mintát szabályozás][throttle].

**Ezeken blokkolása**. Csak adatátviteli ügyfél, mert ez nem jelenti azt ügyfél ártó működött. Csak akkor az ügyfél túllépése miatt a szolgáltatás. De ha egy ügyfél következetesen meghaladja a kvótát, vagy ellenkező esetben úgy viselkedik, helytelen, akkor blokkolhatják őket. Adja meg egy sávon kívüli folyamat felhasználó kérelem első feloldva.

**Vezető választás használata**. Ha egy feladat koordinálására van szüksége, használja [vezető választás] [ leader-election] egy koordinátor kiválasztásához. Ily módon a koordinátor nem a hibaérzékeny pontok kialakulását. A koordinátor nem sikerül, ha egy új van kiválasztva. Helyett egy vezető választás algoritmus teljesen új megvalósításához, fontolja meg egy vásárolt megoldás például Zookeeper.  

**Van a vizsgálat**. A sikeres elérési út minden túl gyakran, jól lett tesztelve azonban nem sikertelen elérési. A rendszer hosszú ideje előtt hiba elérési gyakorolni az éles környezetben futtathat. Van segítségével ellenőrizze a sikertelen, a rendszer a rugalmasság, tényleges sikertelen futtatásától vagy szimulál őket. 

**Chaos mérnöki kihasználásának**. Chaos mérnöki tartalék injektálási fogalmát hibákat vagy olyan rendellenes feltételek véletlenszerűen beszúrva az éles példányok terjeszti ki. 

A strukturált módjáról, hogy az alkalmazások öngyógyító, lásd: [rugalmas alkalmazások az Azure-][resiliency-overview].  

[circuit-breaker]: ../../patterns/circuit-breaker.md
[compensating-transactions]: ../../patterns/compensating-transaction.md
[leader-election]: ../../patterns/leader-election.md
[load-level]: ../../patterns/queue-based-load-leveling.md
[resiliency-overview]: ../../resiliency/index.md
[retry]: ../../patterns/retry.md
[throttle]: ../../patterns/throttling.md
[transient-fault-handling]: ../../best-practices/transient-faults.md

