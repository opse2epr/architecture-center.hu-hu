---
title: Teljesítményre vonatkozó kizárási minták felhőalkalmazásoknál
titleSuffix: Azure Architecture Center
description: Gyakori eljárások, amelyek nagy eséllyel méretezhetőségi problémákat okoznak.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 9ce1177bac2c93139faf6bc757f2866d6b7eac2d
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009729"
---
# <a name="performance-antipatterns-for-cloud-applications"></a>Teljesítményre vonatkozó kizárási minták felhőalkalmazásoknál

A *teljesítményre vonatkozó kizárási minta* egy gyakori eljárás, amely nagy eséllyel eredményez méretezhetőségi problémákat, amikor az alkalmazás nagy terhelésnek van kitéve.

Íme egy gyakori forgatókönyv: Az alkalmazás jól teljesít a teljesítményteszt során. Azután kikerül az éles rendszerre, és elkezd valós számítási feladatokkal foglalkozni. Ezen a ponton elkezd gyengén teljesíteni – elutasítja a felhasználói kéréseket, elakad vagy kivételeket dob. A fejlesztőcsapat két kérdéssel néz szembe:

- Miért nem került elő ez a működési probléma a tesztelés során?
- Hogyan lehet kijavítani?

Az első kérdésre adott válasz egyértelmű. A tesztkörnyezetekben nagyon nehéz a valós felhasználókat, a viselkedésmintáikat és az általuk végzett munkamennyiségeket megfelelően szimulálni. Az egyetlen biztos módja a rendszerek terhelés alatti működésének megértésére, ha éles környezetben vetjük őket vizsgálat alá. Ezzel nem azt állítjuk, hogy nem érdemes teljesítménytesztelést végezni. A teljesítménytesztek kulcsfontosságúak az alapul szolgáló teljesítmény-mérőszámok megállapításához. Emellett azonban az éles környezetben fellépő teljesítményproblémák megfigyelésére és korrigálására is fel kell készülni.

A második kérdés, a probléma megoldása azonban nem ilyen egyértelmű. Itt rengeteg tényező közrejátszhat, és néha a probléma csak bizonyos körülmények között jelentkezik. A rendszerállapot figyelése és a naplózás kulcsfontosságú a kiváltó okok kimutatásához, azonban nem árt tudni, hogy mit keressünk.

A Microsoft Azure-ügyfelekkel folytatott együttműködés során azonosítottunk néhány gyakori teljesítményproblémát, amelyekkel az ügyfelek az éles környezetekben találkoznak. Mindegyik ilyen kizárási minta esetében leírjuk, hogy az adott minta jellemzően miért jelenik meg, mik a tünetei, és milyen technikákkal lehet megoldani a problémát. Emellett kódmintákat is biztosítunk, amelyek bemutatják magukat a kizárási mintákat és azok megoldását.

Egyes kizárási minták a leírásukat olvasva nyilvánvalónak tűnhetnek, mégis sokkal gyakrabban fordulnak elő, mint gondolnánk. Néha előfordul, hogy az alkalmazás kialakítása a helyi környezetben működött, a felhőben azonban nem sikerül megfelelően a méretezése. Az is megtörténhet, hogy az alkalmazás kialakítása kezdetben letisztult és problémamentes, az új funkciók hozzáadásával azonban beférkőzik egy vagy több kizárási minta. Ez az útmutató segíthet azonosítani és korrigálni ezeket a mintákat, bármi legyen is a kiváltó okuk.

Az alábbi lista az általunk azonosított kizárási mintákat sorolja fel:

| Kizárási minta | Leírás |
|-------------|-------------|
| [Foglalt adatbázis][BusyDatabase] | Túl sok feldolgozás van kiszervezve egy adattárba. |
| [Foglalt előtér][BusyFrontEnd] | Az erőforrás-igényes feladatok áthelyezése a háttérszálakra. |
| [Forgalmas I/O][ChattyIO] | Sok kisebb hálózati kérés folyamatos küldése. |
| [Felesleges beolvasások][ExtraneousFetching] | A szükségesnél több adat lekérése, ami szükségtelen I/O működést eredményez. |
| [Nem megfelelő példányosítás][ImproperInstantiation] | Olyan objektumok ismételt létrehozása és megsemmisítése, amelyeket amúgy megosztani és újrahasználni kellene. |
| [Monolitikus adatmegőrzés][MonolithicPersistence] | Egyazon adattár használata nagyon eltérő használati mintákkal rendelkező adatokhoz. |
| [Nincs gyorsítótárazás][NoCaching] | Az adatok gyorsítótárazása nem valósul meg. |
| [Szinkron I/O][SynchronousIO] | A hívó szál blokkolása az I/O végrehajtása során. |

[BusyDatabase]: ./busy-database/index.md
[BusyFrontEnd]: ./busy-front-end/index.md
[ChattyIO]: ./chatty-io/index.md
[ExtraneousFetching]: ./extraneous-fetching/index.md
[ImproperInstantiation]: ./improper-instantiation/index.md
[MonolithicPersistence]: ./monolithic-persistence/index.md
[NoCaching]: ./no-caching/index.md
[SynchronousIO]: ./synchronous-io/index.md
