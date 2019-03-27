---
title: A Materialized View minta
titleSuffix: Cloud Design Patterns
description: Létrehozhat előre kitöltött nézeteket egy vagy több adattár adataiból, ha az adatok formázása nem ideális a szükséges lekérdezési műveletekhez.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 638c4cacfba3f170fdf1c967d783d60c8c6c6b6f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298418"
---
# <a name="materialized-view-pattern"></a>A Materialized View minta

[!INCLUDE [header](../_includes/header.md)]

Létrehozhat előre kitöltött nézeteket egy vagy több adattár adataiból, ha az adatok formázása nem ideális a szükséges lekérdezési műveletekhez. Ez segíthet a hatékony lekérdezésben és adatkinyerésben, valamint az alkalmazás teljesítményének javításában.

## <a name="context-and-problem"></a>Kontextus és probléma

Az adatok tárolásakor a fejlesztők és adat-rendszergazdák gyakran az adatok tárolásának módjára, nem pedig az írás módjára összpontosítanak. A kiválasztott tárolási formátum általában szorosan összefügg az adatok formátumával, az adatok méret- és integritáskezelési követelményeivel és a használt tároló típusával. NoSQL-dokumentumtároló használata esetén például az adatok gyakran szerepelnek összesítések sorozataként, és mindegyik sorozat minden információt tartalmaz az adott entitásról.

Azonban ez negatív hatással lehet a lekérdezésekre. Ha egy lekérdezésnek bizonyos entitások adatainak egy részére van szüksége (például néhány ügyfél rendeléseinek egyes részletei), a kapcsolódó entitás összes adatát ki kell nyernie a szükséges információ beszerzéséhez.

## <a name="solution"></a>Megoldás

A hatékony lekérdezések támogatásához gyakori megoldás egy olyan nézet létrehozása, amely a szükséges eredménykészlethez megfelelő formátumban materializálja az adatokat. A Materialized View minta előre feltöltött adatnézetek létrehozását jelenti az olyan környezetekben, ahol a forrásadatok nem a lekérdezésnek megfelelő formátumban vannak, a megfelelő lekérdezés létrehozása bonyolult, vagy a lekérdezés teljesítménye nem megfelelő az adatok vagy az adattár természete miatt.

Ezek a materializált nézetek, amelyek csak a lekérdezéshez szükséges adatokat tartalmazzák, lehetővé teszik, hogy az alkalmazások gyorsan beszerezzék a számukra szükséges információkat. A táblázatok összekapcsolása és az adatentitások kombinálása mellett a materializált nézetek tartalmazhatják a számított oszlopok vagy adatelemek aktuális értékeit, az értékek kombinálásának vagy adatelemek átalakításának eredményeit, illetve a lekérdezés részeként megadott értékeket is. A materializált nézet egyetlen lekérdezéshez is optimalizálható.

A lényeg az, hogy a materializált nézet és a benne található adatok teljesen pótolhatók, mivel teljesen újraépíthetők a forrásadattárakból. A materializált nézetet soha nem közvetlenül egy alkalmazás frissíti, ezért ez egy specializált gyorsítótár.

Ha a nézet forrásadatai megváltoznak, az új információk megjelenítéséhez frissíteni kell a nézetet. Ezt ütemezheti úgy, hogy automatikusan történjen, vagy amikor a rendszer az eredeti adatok módosítását érzékeli. Egyes esetekben előfordulhat, hogy manuálisan újra létre kell hoznia a nézetet. Az ábra a Materialized View minta használatának példáját tartalmazza.

![Az 1. ábra a Materialized View minta használatának példáját tartalmazza.](./_images/materialized-view-pattern-diagram.png)

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

Hogyan és mikor fog frissülni a nézet. Ideális esetben a forrásadatok változását jelző eseményre adott válaszként újra létrejön a nézet, de ez túlzott terheléshez vezethet, ha a forrásadatok gyorsan változnak. Másik megoldásként fontolja meg egy ütemezett feladat, külső eseményindító vagy manuális művelet használatát a nézet újbóli létrehozásához.

Egyes rendszerekben, például ha az Event Sourcing mintával tartja fenn az adatokat módosító események tárolóját, szükségesek a materializált nézetek. Előfordulhat, hogy az eseménytár információi csak úgy szerezhetők be, ha előre feltölti a nézeteket az aktuális állapot meghatározásához. Ha nem használ Event Sourcing mintát, fontolja meg, hogy a materializált nézet hasznos-e. A materializált nézetek általában kifejezetten egyetlen vagy kis számú lekérdezéshez vannak igazítva. Ha sok lekérdezést használ, a materializált nézetek túlzott tárkapacitási követelményeket és tárolási költségeket eredményezhetnek.

Vegye figyelembe az adatkonzisztenciára gyakorolt hatást a nézet létrehozásakor, illetve ütemezett frissítések használatakor. Ha a forrásadatok változnak a nézet létrehozásakor, az adatok nézetben található másolata nem lesz teljesen konzisztens az eredeti adatokkal.

Fontolja meg, hol fogja tárolni a nézetet. A nézetnek nem kell ugyanazon a tárolóban vagy partíción lennie, mint az eredeti adatoknak. Több különböző partíció részeiből is származhat.

A nézet újraépíthető, ha elveszik. Ezért ha a nézet átmeneti, és csak a lekérdezési teljesítmény vagy a méretezhetőség javítására szolgál az adatok aktuális állapotának megjelenítésével, akkor tárolható egy gyorsítótárban vagy egy kevésbé megbízható helyen.

Materializált nézetek meghatározásakor maximalizálhatja a nézet értékét, ha a meglévő adatelemek számításának vagy átalakításának alapján, a lekérdezésben átadott értékek alapján, vagy ahol lehetséges, ezen értékek kombinációjának alapján adatelemeket vagy oszlopokat ad hozzá.

Ahol a tárolási mechanizmus támogatja, fontolja meg a materializált nézet indexelését a teljesítmény további növelése érdekében. A legtöbb relációs adatbázis támogatja a nézetek indexelését, ahogy az Apache Hadoopon alapuló big data-megoldások is.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- Materializált nézetek létrehozása olyan adatok helyett, amelyeket nehéz közvetlenül lekérdezni, vagy ahol a lekérdezéseknek nagyon összetettnek kell lenniük a normalizált, részben strukturált vagy strukturálatlan módon tárolt adatok kinyeréséhez.
- Ideiglenes nézetek létrehozása, amelyek jelentősen javíthatják a lekérdezés teljesítményét, vagy közvetlenül a felhasználói felület forrásnézeteiként vagy adatátviteli objektumaiként működhetnek jelentéskészítéshez vagy megjelenítéshez.
- Átmenetileg csatlakoztatott vagy leválasztott forgatókönyvek támogatása, ahol nem mindig lehet csatlakozni az adattárhoz. Ebben az esetben a nézet helyileg gyorsítótárazható.
- Lekérdezések egyszerűsítése és adatok felfedése kísérletezéshez anélkül, hogy szükség lenne a forrásadatok formátumának ismeretére. Ilyen eset például, ha egy vagy több adatbázisban található táblákat, illetve egy vagy több NoSQL-tárolóban található tartományokat kapcsol össze, majd formázza az adatokat a kívánt felhasználásnak megfelelően.
- A forrásadatok adott részhalmazaihoz való hozzáférés biztosítása, amelyeket biztonsági vagy adatvédelmi okokból nem kíván általános elérhetővé, módosíthatóvá vagy a felhasználók által láthatóvá tenni.
- Különböző adattárolók áthidalása az egyes tárolók képességeinek kihasználása érdekében. Ilyen eset például, ha materializált nézetek tárolására kíván használni egy felhőalapú tárolót, amely referencia-adattárként hatékony az írásban, valamint egy relációs adatbázist, amely jó lekérdezési és olvasási teljesítményt kínál.

A minta használata a következő esetekben nem hasznos:

- A forrásadatok egyszerűek és könnyen lekérdezhetők.
- A forrásadatok gyorsan változnak vagy nézet használata nélkül is elérhetők. Ezekben az esetekben kerülje el a nézetek létrehozásával járó többletterhelést.
- A konzisztencia kiemelt fontosságú. Előfordulhat, hogy a nézetek nem teljesen konzisztensek az eredeti adatokkal.

## <a name="example"></a>Példa

Az alábbi ábra egy példát mutat be egy értékesítési összesítés létrehozására a Materialized View minta használatával. Az Azure Storage-tárfiók különböző partícióin lévő Order, OrderItem és Customer táblázatokban található adatokat kombinálja egy olyan nézet létrehozásához, amely tartalmazza az Elektronikai termékek kategória minden termékének teljes értékesítési értékét, valamint az egyes elemeket megvásárló ügyfelek számát.

![2. ábra: A Materialized View minta használata értékesítési összesítés létrehozására](./_images/materialized-view-summary-diagram.png)

Ennek a materializált nézetnek a létrehozásához összetett lekérdezések szükségesek. Ha azonban materializált nézetként teszi elérhetővé a lekérdezés eredményét, a felhasználók könnyen beszerezhetik az eredményeket, és közvetlenül felhasználhatják, illetve belefoglalhatják őket egy másik lekérdezésbe. A nézetet valószínűleg egy jelentéskészítési rendszerben vagy irányítópulton fogják használni, és ütemezés szerint (például hetente) frissíthető.

> Bár ebben a példában az Azure Table Storage-ot használjuk, számos más relációsadatbázis-kezelő rendszer is natív támogatást nyújt a materializált nézetekhez.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). A materializált nézet összegző információit karban kell tartani, hogy az alapul szolgáló adatok értékeit tükrözze. Az adatértékek változásával nem biztos, hogy praktikus az összegző adatok valós idejű frissítése, és olyan módszert kell helyette alkalmazni, amely végül konzisztens adatokat eredményez. A cikk összefoglalja az elosztott adatok konzisztenciájának megőrzésével kapcsolatos problémákat, és bemutatja a különböző konzisztenciamodellek előnyeit és hátrányait.
- [Command and Query Responsibility Segregation (CQRS) minta](./cqrs.md). A materializált nézetben lévő információk frissítésére szolgál úgy, hogy válaszol az alapul szolgáló adatok módosításakor fellépő eseményekre.
- [Event Sourcing minta](./event-sourcing.md). Használja a CQRS mintával együtt a materializált nézetben található információk karbantartásához. Ha a materializált nézet alapját képző adatértékek változnak, a rendszer ezeket a változásokat leíró eseményeket indíthat, és mentheti őket egy eseménytárban.
- [Index Table minta](./index-table.md). A materializált nézet adatainak rendszerezése általában egy elsődleges kulcs szerint történik, de előfordulhat, hogy a lekérdezéseknek másik mezőkben lévő adatok vizsgálatával kell lekérniük az információkat ebből a nézetből. A használatával másodlagos indexek hozhatók létre az adatkészletekből olyan adattárakban, amelyek nem támogatják a natív másodlagos indexeket.
