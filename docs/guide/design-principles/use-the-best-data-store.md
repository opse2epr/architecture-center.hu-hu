---
title: Használja a feladathoz legmegfelelőbb adattárat
titleSuffix: Azure Application Architecture Guide
description: Válassza az adataihoz és azok felhasználásához leginkább megfelelő tárolótechnológiát.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2f83e7a184e8fa94f49da4e8fd5252396ecfd632
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298522"
---
# <a name="use-the-best-data-store-for-the-job"></a>Használja a feladathoz legmegfelelőbb adattárat

## <a name="pick-the-storage-technology-that-is-the-best-fit-for-your-data-and-how-it-will-be-used"></a>Válassza az adataihoz és azok felhasználásához leginkább megfelelő tárolótechnológiát

Manapság már nem tennénk fel az összes adatunkat egy hatalmas relációs SQL-adatbázisba. A relációs adatbázisok jól végzik a dolgukat – garantálják az ACID (atomitás, konzisztencia, izoláció, tartósság) megvalósulását a relációs adatokkal használt tranzakciókhoz. Ezek azonban költségekkel járnak:

- A lekérdezésekhez költséges összekapcsolásokra lehet szükség.
- Az adatokat normalizálni kell, és illeszkedniük kell egy előre meghatározott sémához (íráskor meghatározott séma).
- A zárolási verseny hatással lehet a teljesítményre.

Bármely nagy megoldásról legyen szó, egyetlen adattár-technológia jó eséllyel nem fog megfelelni az összes igénynek. A relációs adatbázisok alternatívái lehetnek például a kulcs/értéktárak, a dokumentum-adatbázisok, a keresőmotor-adatbázisok, az idősorozat-adatbázisok, az oszlopcsalád-adatbázisok, és a gráfadatbázisok. Mindnek vannak előnyei és hátrányai, és a különféle típusú adatok jobban illeszthetők az egyik vagy a másik fajtához.

Például egy termékkatalógust tárolhat egy dokumentum-adatbázisban, (például Cosmos DB), amely rugalmas sémát tesz lehetővé. Ebben az esetben minden egyes termékleírás egy önálló dokumentumot képez. A teljes katalógusra kiterjedő lekérdezések esetében indexelheti a katalógust, az indexet pedig az Azure Search-ben tárolhatja. A termékleltár kerülhet egy SQL adatbázisba, mivel ezeknek az adatoknak ACID-garanciára van szüksége.

Ne feledje, hogy adat alatt nem csak a megőrzött alkalmazásadatokat értjük. Ide tartoznak az alkalmazásnaplók, események, üzenetek és a gyorsítótárak is.

## <a name="recommendations"></a>Javaslatok

**Ne relációs adatbázissal oldjon meg mindent**. Ha szükséges, gondolkozzon más adattárakban is. Lásd: [A megfelelő adattároló kiválasztása][data-store-overview].

**Vezesse be a többnyelvű adatmegőrzést**. Bármely nagy megoldásról legyen szó, egyetlen adattár-technológia jó eséllyel nem fog megfelelni az összes igénynek.

**Vegye figyelembe az adatok típusát**. Például helyezze a tranzakciós adatokat SQL-be, a JSON-dokumentumokat egy dokumentum-adatbázisba, a telemetriai adatokat egy idősorozat-adatbázisba, az alkalmazásnaplókat az Elasticsearch-be, a blobokat pedig az Azure Blob Storage-ba.

**Részesítse előnyben a rendelkezésre állást az (erős) konzisztenciával szemben**. A CAP-tétel szerint az elosztott rendszerek esetén mindenképpen kompromisszumot kell kötni a rendelkezésre állás és a konzisztencia között. (A CAP-tétel másik alapját képező hálózati partíciókat soha nem lehet teljes mértékben elkerülni.) Sok esetben nagyobb rendelkezésre állás érhető el egy *végleges konzisztencia* modell bevezetésével.

**Vegye figyelembe a fejlesztői csapat szaktudását**. Habár a többnyelvű adatmegőrzés használata előnyökkel jár, túlzásokba lehet esni. Egy új adattárolási technológia bevezetéséhez új szaktudásra van szükség. A fejlesztői csapatnak értenie kell a technológia minél hatékonyabb működtetéséhez. Ismerniük kell a megfelelő használati mintákat, tudniuk kell optimalizálni a lekérdezéseket, nagy teljesítményre hangolni a rendszert stb. Vegye ezt is figyelembe, amikor tárolási technológiát választ.

**Kompenzáló tranzakciók használata**. A többnyelvű adatmegőrzés egyik mellékhatásaként egy tranzakció több tárolóba is írhat adatot. Ha hiba lép fel, kompenzáló tranzakciók használatával vonhatja vissza a már végrehajtott lépéseket.

**Keresse meg a körülhatárolt kontextusokat**. A *körülhatárolt kontextus* a szakterület-vezérelt tervezés egyik fogalma. A körülhatárolt kontextus egy szakterületi modell körüli explicit határ, amely megadja, hogy a modell a szakterület mely részeire vonatkozik. A körülhatárolt kontextus ideális esetben az üzleti szakterület egy részterületét képezi le. A rendszerben található körülhatárolt kontextusok tökéletes helyek arra, hogy bevezesse a többnyelvű adatmegőrzést. Például a „termékek” megjelenhetnek egyszerre a termékkatalógus és a termékleltár részterületen, viszont a két részterületen a termékek esetén jó eséllyel más-más tárolási, frissítési és lekérdezési igények merülnek fel.

[data-store-overview]: ../technology-choices/data-store-overview.md
