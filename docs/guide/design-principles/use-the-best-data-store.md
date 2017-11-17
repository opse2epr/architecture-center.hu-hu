---
title: "A feladat a legjobb adattár használata"
description: "Válassza ki a tárolási technológia, amely a legjobb térkihasználás érdekében az adatokat, és hogyan fogja használni"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: ef9439f7a3766d13b498eac915e0f5afd23de4e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="use-the-best-data-store-for-the-job"></a>A feladat a legjobb adattár használata

## <a name="pick-the-storage-technology-that-is-the-best-fit-for-your-data-and-how-it-will-be-used"></a>Válassza ki a tárolási technológia, amely a legjobb térkihasználás érdekében az adatokat, és hogyan fogja használni

A nap, ha meg szeretné csak odatapadjon az összes adat egy nagy relációs SQL adatbázisba már meg van. Relációs adatbázisok a következők: Ezek nagyon jó &mdash; sav biztosító biztosítja, hogy az egyes tranzakciókra vonatkozóan relációs adatok. Azonban néhány költséggel származnak:

- Lekérdezések költséges illesztések lehet szükség.
- Adatok normalizálni kell, és egy előre meghatározott schema (séma írás) felel meg.
- Zárolási versenyt hatással lehet a teljesítményre.

Nagy megoldás valószínű, hogy a tároló egyetlen technológiát nem töltse ki az igényeinek. Relációs adatbázisok alternatívák például kulcs-érték tárolók, dokumentum-adatbázisokat, keresési adatbázisok, idő adatsorozat adatbázisok, oszlop családba tartozó adatbázisok, és graph adatbázisok. Minden egyes vannak előnyei és hátrányai, és különböző típusú adatok sorolhatók hatnak egy vagy egy másikra. 

Például egy dokumentum-adatbázisban, például Cosmos DB, ami lehetővé teszi a rugalmas séma termékkatalógus előfordulhat, hogy tárolják. Ebben az esetben minden egyes termék leírása egy önálló-dokumentumot. A lekérdezések teljes keresztül előfordulhat, hogy a katalógus index, és tárolja az index az Azure Search. Termék mehet az SQL-adatbázisba, mert adatok szükséges ACID garanciát.

Ne feledje, hogy nem csak a megőrzött alkalmazásadatok szerepel. Alkalmazásnaplók, az események, az üzenetek és a gyorsítótárak is tartalmaz.

## <a name="recommendations"></a>Javaslatok

**Ne használjon egy relációs adatbázisban minden**. Fontolja meg az egyéb adattárakhoz, amikor szükséges. Lásd: [válassza ki a megfelelő tárolót][data-store-overview].

**Polyglot adatmegőrzési kihasználásának**. Nagy megoldás valószínű, hogy a tároló egyetlen technológiát nem töltse ki az igényeinek. 

**Vegye figyelembe az adatok**. Például helyezték SQL, helyezze a JSON-dokumentumok egy dokumentum adatbázisba, telemetriai adatokat helyezett adatok kiinduló, alkalmazásnaplók be Elasticsearch, és hogy a BLOB az Azure Blob Storage idősor tranzakciós adatok.

**Rendelkezésre állási inkább keresztül (erős) konzisztencia**. A Tengelysapka tétel azt jelenti, hogy egy elosztott rendszer rendelkezésre állás és a konzisztencia közötti kompromisszumot kell hoznia. (Hálózati partíciók, a többi szakasza CAP tétel soha nem teljesen elkerülhető.) Gyakran, érhető el magas rendelkezésre állás érdekében elfogadásával egy *végleges konzisztencia* modell. 

**Vegye figyelembe a fejlesztői csapat szakértelem készlete**. Polyglot adatmegőrzési használatának előnye van, de lehetséges a Visszalépés visszanyerésében. Egy új tárolási technológiát bevezetése meg kell adni egy új képességek. A fejlesztői csapat tisztában kell a technológiát minél hatékonyabb működtetését. Ismerniük kell a megfelelő használati szokásokról, optimalizálása lekérdezi, a teljesítmény hangolására és így tovább. Figyelembe ezt a annak eldöntéséhez, hogy a tárolási technológiákat. 

**Tranzakciók compensating használata**. Egyik mellékhatása polyglot adatmegőrzési, hogy egy tranzakció lehet adatokat írni több üzletben. Ha valami nem sikerül, használja a kompenzációs tranzakciók visszavonása minden lépést, amely már befejeződött.

**Nézze meg a kötött környezetek**. *A kötött környezetben* van egy kifejezés tervezési folyamat tartományból. A kötött környezetben egy explicit határ tartománymodell körül, és határozza meg, a tartomány mely részei a modell vonatkozik. Ideális esetben a vállalati tartomány altartomány kötött kontextusban rendeli hozzá. A rendszer a kötött környezetek kell figyelembe venni polyglot adatmegőrzési természetes helyen. Például "termékek" jelenhetnek meg a termékkatalógus altartomány és a termék altartomány is, de nagyon valószínű, hogy a két altartományok tárolás, frissítése és termékek lekérdezése különböző követelményekkel rendelkezik-e.

[data-store-overview]: ../technology-choices/data-store-overview.md