---
title: Terv módosítása
description: Egy fokozatosan terv folyamatos innováció kulcsa.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 35e91228f3fb0a303594ec06f05b6865008e3a4f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-evolution"></a>Alakulása kialakítása

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>Egy fokozatosan kialakítása esetén folyamatos innováció kulcsa

Minden sikeres alkalmazások módosítása keresztül arra, hogy javítsa ki a hibákat, új szolgáltatásokkal, új technológiák állapotba kerüljön-e vagy hogy a meglévő rendszerek jobban méretezhető és rugalmas. Ha az alkalmazás minden részén szorosan összekapcsolt válik nagyon nehéz bevezetéséhez a módosításokat a rendszer. Az alkalmazás egy részét a módosítják a törés egy másik, vagy következtében a módosítások a teljes codebase keresztül ripple.

Ez a probléma az egységes alkalmazások nem korlátozódik. Az alkalmazás kell bontja fel a rendszer az szolgáltatásaiba, de még megfigyelhető, hogy a rendszer kemény és rideg szoros kapcsoló rendezést. Azonban ha szolgáltatást tervezték, hogy fejlődnek, csoportok innovációját annak, és folyamatosan a új szolgáltatások biztosításához. 

Mikroszolgáltatások egy evolutonary terv eléréséhez népszerű úgy egyre, mivel azok cím, az itt felsorolt szempontok számos.

## <a name="recommendations"></a>Javaslatok

**Magas Kohéziós és elveszíteni kapcsoló**. A "szolgáltatás" *javul* Ha tartozó logikailag egymáshoz funkciókat biztosít. Szolgáltatások *lazán összekapcsolt* Ha egy szolgáltatás módosíthatja a másik módosítása nélkül. Magas Kohéziós általában azt jelenti, hogy egy függvény változásai szükséges-e más kapcsolódó funkciók változásait. Ha talál meg, hogy a szolgáltatás frissítése igényel koordinált más szolgáltatások frissítéseket, lehet a jele, hogy a szolgáltatások nincsenek-e javul. A kitűzött célokat, tartomány-központú kialakítás (nnn) egyik identitás ezeken a határokon belül.

**Tartomány Tudásbázis beágyazására**. Amikor egy ügyfél a szolgáltatás használ fel, a tartomány az üzleti szabályok érvényesítési felelősséget nem tartozik az ügyfélen. Ehelyett a szolgáltatás kell foglalják magukban a tartomány Tudásbázis saját felelősségére eső összes. Ellenkező esetben minden ügyfélnek van alkalmazva az üzleti szabály, és végül az alkalmazás különböző részei elosztva tartomány ismeretekkel. 

**Használja a aszinkron üzenetkezelési**. Aszinkron üzenetkezelési módja a használata leválasztja az üzenet létrehozója az ügyféltől. A küldőnek nem függ a fogyasztó válaszol az üzenethez, vagy bármely adott műveletek. A közzétételi/sub architektúra a gyártó előfordulhat, hogy nem fogják tudni ki van fel az üzenetet. Új szolgáltatások könnyen felhasználhat az üzenetek a termelő módosítás nélkül.

**Az átjáró tartomány Tudásbázis nem build**. Átjárók mikroszolgáltatások architektúra, többek között a kérés útválasztási, protokollfordításhoz, terheléselosztást vagy hitelesítési számára hasznos lehet. Azonban az átjáró kell lennie az ilyen infrastructure funkciót. Bármely tartomány Tudásbázis nehéz függőség váljon elkerülése érdekében érdemes valósítja meg.

**Megnyitás felületek elérhetővé**. Ne hozzon létre egyéni fordítási rétegek, amelyek a szolgáltatások között elhelyezkedő. Ehelyett egy szolgáltatást egy API-t egy jól meghatározott API-szerződés kell tenni. Az API-t kell látható el verzióadatokkal, úgy, hogy az előző verziókkal való kompatibilitás megőrzése is fejleszteni a az API-t. Ily módon frissítheti a szolgáltatás az összes fölérendelt tőle függő szolgáltatások frissítéseket koordináló nélkül. Nyilvános irányuló szolgáltatások elérhetővé kell tenni egy RESTful API HTTP Protokollon keresztül. Háttér-szolgáltatások használhatja egy RPC-style üzenetküldési protokoll jobb teljesítmény érdekében. 

**Tervezési és a szolgáltatási szerződések vizsgálatot**. Szolgáltatások elérhetővé tenni a jól meghatározott API-k, amikor is létrehozhat és második megvizsgál ezen API-k. Ily módon is létrehozhat és tesztelése egy forgó függő szolgáltatásai mentése nélkül szolgáltatás. (Természetesen volna továbbra is hajtsa végre az integráció és betöltése, szemben a valódi szolgáltatások tesztelése.)

**Absztrakt infrastruktúra tartomány programot innen máshová**. Nem lehetővé teszik az beszerzése keverve infrastruktúrával kapcsolatos funkciókat, például üzenetküldési és az adatmegőrzési tartomány logika. Ellenkező esetben a tartományi logika fog módosításához frissítések az infrastruktúra rétegekhez, és ez fordítva is igaz. 

**Több területet aggályokat egy külön szolgáltatásra kiszervezése**. Például ha számos szolgáltatás-kérelmek hitelesítéséhez szükséges, mozgathatja ezt a funkciót a saját szolgáltatásba. Ezután a hitelesítési szolgáltatás sikerült fejlődnek &mdash; például egy új hitelesítési folyamat hozzáadásával &mdash; , az azt használó szolgáltatások bármelyike érintése nélkül.

**Szolgáltatások telepítése függetlenül**. Ha a DevOps team is függetlenül egyéb szolgáltatásokat az alkalmazásban egy szolgáltatás telepítéséhez frissítések gyorsan és biztonságosan akkor fordulhat elő. Hibajavításokat tartalmaz, és új funkciók is állítható, a több rendszeres ütemben történik. Az alkalmazás és a a kiadási folyamat támogatására független frissítések megtervezése.