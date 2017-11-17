---
title: "Materializált nézet"
description: "Az adatokat egy vagy több adattárolókhoz keresztül előfeltöltött nézetek létrehozása az adatok nem ideális formázott kötelező lekérdezési műveletek."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 992abcb57204c65a7ca9e9e2525d3ea7339c4a2c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="materialized-view-pattern"></a>Materializált nézet minta

[!INCLUDE [header](../_includes/header.md)]

Az adatokat egy vagy több adattárolókhoz keresztül előfeltöltött nézetek létrehozása az adatok nem ideális formázott kötelező lekérdezési műveletek. A támogatási hatékony lekérdezésére és az adatok kinyerése érdekében, és az alkalmazások teljesítményének javítása.

## <a name="context-and-problem"></a>A környezetben, és probléma

Adatok tárolására, a prioritás a fejlesztők és az adatok a rendszergazdák gyakran arra irányul, hogy az adatok tárolási módjára, hogyan írásvédett szemben. A kiválasztott tárolási formátum általában szorosan összefügg az adatok, az adatok mérete és sértetlenségét és milyen típusú használt tároló kezeléséhez szükséges formátumnak. Például NoSQL dokumentum tároló használata esetén az adatok gyakran képviselt összesítések, sorozataként tartalmazó minden olyan információt, hogy az entitás.

Azonban ez is negatív hatással a lekérdezések. Ha a lekérdezés csak néhány entitások, például több ügyfelek nem minden a rendelés részleteit rendelések összefoglalását az adatok egy részét az összes szempontjából fontos entitásokra vonatkozó adatot ahhoz, hogy a szükséges adatokat az beszerzése kell kibontásához.

## <a name="solution"></a>Megoldás

Hatékony lekérdezése támogatásához egy közös megoldás létrehozásához, előre megvalósul a szükséges eredmények készlet megfelelő formátumú a adatokat megjelenítő. A materializált nézet mintát ismerteti az adatok előfeltöltött nézetek generálása környezetekben, ahol az adatok nem a megfelelő formátumú lekérdezése, ahol megfelelő lekérdezés létrehozása nehéz, vagy ha lekérdezési teljesítmény gyenge jellemzői miatt a adatok vagy az adattárban.

A materializált nézeteket, amelyek csak tartalmaz adatot, egy lekérdezés által igényelt, lehetővé teszi a alkalmazások gyorsan a szükséges információk beszerzéséhez. Mellett tábla, vagy kombinálja az entitások, materializált nézetek a számított oszlopok vagy adatelemek, az értékek összefűzése vagy átalakítások végrehajtása az elemeket, és a lekérdezés részeként megadott értékek eredményeit az aktuális értékek tartalmazhatnak . A materializált nézet is optimalizálhatók a csupán egyetlen lekérdezést.

A lényeg az, hogy egy materializált nézet és a benne található adatok teljes rendelkezésre álló mert akkor is teljes mértékben építhető újra, a forrás adatok áruházakból. A materializált nézet soha nem frissít közvetlenül egy alkalmazás, és azt egy speciális gyorsítótár.

A forrás az adatok a nézet a nézet meg kell adni az új információval. Ütemezheti az Ez automatikusan megtörténjen-e, vagy amikor a rendszer észleli, hogy az eredeti adatok. Egyes esetekben szükség lehet manuálisan újragenerálja a nézetet. Az ábra előfordulhat, hogy hogyan használható a materializált nézet mintát példáját mutatja be.

![1. ábra szemlélteti, hogyan lehet, hogy használni a materializált nézet minta](./_images/materialized-view-pattern-diagram.png)


## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

Hogyan és mikor frissíti a nézetet. Ideális esetben azt kell generálja újra egy módosítva lett a forrásadatok jelző esemény bekövetkeztekor bár ez vezethet túlzott terhelés, ha az adatok gyors megváltozik. Azt is megteheti érdemes lehet egy ütemezett feladatot, egy külső eseményindító vagy a manuális műveletet újragenerálja a nézetet.

Az egyes rendszerek, például ha az esemény-forrás használatával mintát csak azokat az eseményeket az adatokat módosító tárolhatják a materializált nézetek szükség. Előzetes megadása a nézetek az aktuális állapot meghatározásához minden események vizsgálatával, előfordulhat, hogy az egyetlen lehetőség a esemény áruházból információkhoz. Ha esemény forrás nem használja, fontolja meg, hogy a materializált nézet hasznos vagy nem szeretné. Az egyes materializált nézetek egy vagy néhány lekérdezések igazíthatók. Sok lekérdezések használata esetén materializált nézetek eredményezhet elfogadhatatlan tárolókapacitási követelmények és a tárolási költségű.

Vegye figyelembe az adatok konzisztenciájának gyakorolt, a nézet létrehozásakor, ha a nézet frissítése, ha ez az ütemezés szerint történik. Az adatok a ponton módosítani, ha a nézet jön létre, a nézetben az adatok másolatát nem fogja teljesen konzisztensek legyenek az eredeti adatokkal.

Fontolja meg, ahol a nézetet fogja tárolni. A nézet nem rendelkezik az ugyanarra a tároló vagy az eredeti adatokkal partíció található. A kombinált néhány különböző partíciók részhalmaza lehet.

Nézet úgy is, ha elveszett. Miatt, amely ha a nézet átmeneti, és csak akkor használja, a lekérdezési teljesítmény javítása a tükrözik az adatok aktuális állapotát, vagy, méretezhetőség javítása érdekében tárolható a gyorsítótár vagy kevésbé megbízható helyen.

A materializált nézet meghatározásakor maximalizálása érdekében hozzáadását adatelemek vagy oszlopok alapján számítási vagy átalakítási meglévő adatelemek, a lekérdezés az átadott értékeket, vagy adott esetben ezek az értékek kombinációi értéke.

A teljesítmény növelése érdekében további materializált nézet indexelje, ahol a tárolási mechanizmus támogatja azt. A legtöbb relációs adatbázisok nézetek, indexelő is támogatják alapján Apache Hadoop big data-megoldások.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában akkor hasznos, ha:
- Nézetek létrehozása materializált adatok, amelyek nehéz lekérdezése közvetlenül, vagy ha lekérdezések nagyon összetett normalizált, félig strukturált vagy strukturálatlan módon tárolt adatokat nyerhet ki kell lennie.
- Amely jelentősen javíthatja a lekérdezések teljesítményét, vagy működhet közvetlenül forrás nézet, és az adatok átvitel objektumok a felhasználói felület, a jelentési vagy a megjelenítési ideiglenes nézetek létrehozása.
- Alkalmanként támogató csatlakoztatott vagy nincs csatlakoztatva a forgatókönyvek, ahol az adattárolóhoz kapcsolat nem mindig érhető el. A nézet gyorsítótárazható helyileg ebben az esetben.
- Lekérdezések egyszerűsítése, és úgy, hogy nincs szükség a Forrás-adatformátum ismerete kísérletezhet az adatok. Például különböző tábla egy vagy több adatbázist, vagy egy vagy több tartományt a NoSQL-tárolókon, és majd a a az adatok az esetleges formázási használják.
- Való hozzáférés biztosításához a forrásadatok meghatározott fájlcsoportokat, hogy a biztonsági vagy adatvédelmi okokból, nem lehet általánosan elérhető, nyissa meg a módosítása, vagy teljesen közvetlenül a felhasználók számára.
- Adatközponthíd-képzés különböző adatokat tárolja, az egyes képességek előnyeit. Használata esetén például a felhőben levő tárolójának, amely hatékony írásra, a referencia-tárolót, és egy relációs adatbázisban, amely jó lekérdezés és a materializált nézetek tárolásához olvasási teljesítmény kínál.

Ez a minta nem a következő esetekben lehet hasznos:
- Az adatok egyszerű és lekérdezés könnyen.
- Az adatok nagyon gyorsan a változtatásokat, illetve egy nézet használata nélkül is elérhetők. Ezekben az esetekben ne a feldolgozási terhelést növelni az nézetek létrehozása.
- Konzisztencia egy magas prioritású virtuális gép. A nézetek nem mindig lehet az eredeti adatok teljes mértékben egységes.

## <a name="example"></a>Példa

Az alábbi ábra a materializált nézet minta használatával az értékesítési összegzését létrehozásához példáját mutatja be. Az Azure-tárfiók külön partíciók a sorrendben, OrderItem és ügyfél-táblázatok adatait a rendszer kombinálja a teljes értékesítési értéke az egyes termékek Electronics kategóriában, valamint végző ügyfelek számát tartalmazó nézet létrehozásához az egyes elemek vásárlások.

![2. ábra: A materializált nézet minta használatával az értékesítési összegzését létrehozásához](./_images/materialized-view-summary-diagram.png)


A materializált nézet létrehozásához szükséges összetett lekérdezések. Azonban jelentkezik, mintha a lekérdezés eredménye egy materializált nézet, a felhasználók könnyen az eredmények és használhatja őket közvetlenül vagy bele őket egy másik lekérdezést. A nézet valószínűleg jelentéskészítési rendszeren és irányítópultot használ, és frissíthetik például heti ütemezés szerint.

>  Bár ebben a példában az Azure table storage használja, sok relációs adatbázis-kezelő rendszerek is támogatást nyújt a natív materializált nézetekhez.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:
- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). Az összefoglaló információkat a materializált nézet rendelkezik, hogy azok megfeleljenek az alapul szolgáló adatértékek fenntartásához. Az adatok értékek módosítása, nem lehet frissíteni a valós idejű összefoglalókat gyakorlati, és helyette konfigurálnia kell egy idővel konzisztenssé módszert is fogad el. Összesíti a kérdésekkel karbantartása konzisztencia elosztott adatokon keresztül, és ismerteti az előnyöket és a különböző konzisztencia modellek kompromisszumot.
- [A parancs és a lekérdezés felelősségi elkülönítése (CQRS) minta](cqrs.md). Használja a információinak frissítése a materializált nézet által válaszol az eseményeket, amelyek akkor történik, ha az alapul szolgáló adatértékek módosítása.
- [Esemény Sourcing mintát](event-sourcing.md). Együttes használatával a CQRS mintával materializált nézet adatainak kezelése. Az adatértékek materializált nézet alapuló módosításakor a rendszer merülhet események leíró ezeket a módosításokat, és mentse őket egy esemény áruházban.
- [Index táblázat mintát](index-table.md). A materializált nézet adatainak általában egy elsődleges kulcs szerint vannak rendezve, de lekérdezések módosítania kell a nézet más mezők megvizsgálásával adatok lekérését. Hozhat létre másodlagos indexek adatkészleteket, amelyek nem támogatják natív másodlagos indexek adattárak használatát.
