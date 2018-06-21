---
title: Index táblázat
description: Indexek létrehozása gyakran lekérdezések által hivatkozott adattárolókhoz mezőinek keresztül.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 24a1061349af84d13f05f88a1698b4efe4b0f449
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541785"
---
# <a name="index-table-pattern"></a>Index táblázat minta

[!INCLUDE [header](../_includes/header.md)]

Indexek létrehozása gyakran lekérdezések által hivatkozott adattárolókhoz mezőinek keresztül. Ebben a mintában lekérdezés jobb teljesítmény érdekében az alkalmazások gyorsabban keresse meg az adatok lekérése a tárolóban történő engedélyezése.

## <a name="context-and-problem"></a>A környezetben, és probléma

Sok adattárolókhoz rendezheti az adatokat az elsődleges kulcs használatával entitások gyűjteményét. Egy alkalmazás ezt a kulcsot segítségével keresse meg és adatok lekérdezéséhez. Az ábra ügyféladatok okozó adattárat példáját mutatja be. Az elsődleges kulcs, az ügyfél-azonosító. Az ábrán látható ügyfél adatait az elsődleges kulcsot (ügyfél-azonosító) szerint vannak rendezve.

![1. ábra - ügyfél adatait az elsődleges kulcsot (ügyfél-azonosító) szerint vannak rendezve](./_images/index-table-figure-1.png)


Míg az elsődleges kulcs értékes adatlehívás, ennek a kulcsnak az értéke alapján lekérdezésekhez, az alkalmazás nem feltétlenül tudni használni az elsődleges kulcs, ha néhány többi mező alapján adatainak beolvasása. Az ügyfelek a példában az alkalmazás nem használható a felhasználói azonosító elsődleges kulcs beolvasásához ügyfelek Ha néhány attribútum a város, ahol az ügyfél például értéke kizárólag Vezérlőpultjának adatokat lekérdezi. Ez például-lekérdezést végrehajtani, az alkalmazás beolvasásához, és vizsgálja meg a minden felhasználói rekord, amely lassú folyamat sikerült előfordulhat, hogy rendelkezik.

Számos relációs adatbázis-felügyeleti rendszer támogatja a másodlagos indexek. Egy másodlagos index egy külön adatszerkezet, amely egy vagy több nonprimary (másodlagos) kulcs mező szerint van rendezve, és azt jelzi, hogy minden egyes indexelt értékre adatok tárolására. Egy másodlagos index elemeinek általában a másodlagos kulcsok ahhoz, hogy az adatok gyors keresési érték szerint vannak rendezve. Ezek az indexek általában automatikusan kezeli az adatbázis-kezelő rendszer által.

Tetszőleges számú másodlagos indexek csak szeretne, amely az alkalmazás végrehajtja a különböző lekérdezéseket hozhat létre. Például egy ügyfél tábla egy relációs adatbázisban, ahol az ügyfél-azonosító-e az elsődleges kulcs, célszerű egy másodlagos index hozzáadása a Város mezőre az, ha az alkalmazás által a város, hol találhatók azok az ügyfelek gyakran keres.

Azonban annak ellenére, hogy másodlagos indexek közös relációs rendszerekben, felhőalapú alkalmazások által használt legtöbb NoSQL adattároló nem ad meg egy egyenértékű szolgáltatás.

## <a name="solution"></a>Megoldás

Az adattár nem támogatja a másodlagos indexek, ha azokat manuálisan emulálni saját index táblák létrehozásával. Az index táblázat által a megadott kulcs rendezi az adatokat. Három stratégiák gyakran használják az index táblázat szükséges másodlagos indexek száma és a lekérdezések egy alkalmazás végző jellegétől függően rendszerezésére szolgál.

Az első stratégia, hogy minden index tábla duplikált, de szervezheti, különböző kulccsal (teljes denormalization). Az alábbi ábrán látható város és a Vezetéknév ügyfélinformációkat rendszerezése index táblákhoz.

![2. ábra - adatok ismétlődik minden index tábla](./_images/index-table-figure-2.png)


Ezt a stratégiát, ha az adatok viszonylag statikus minden kulcs segítségével kérnek tőle hányszor képest. Ha az adatok több dinamikus, a feldolgozási terhelést növelni az egyes index táblázatot karbantartó túl nagyra nő esetében ez a megközelítés hasznos lehet. Ha az adatok mennyisége túl nagy, az ismétlődő adatok tárolásához szükséges lemezterület mennyisége is jelentős.

A második stratégia, ha a normalizált index táblák szerint vannak rendezve különböző kulcsokat és az eredeti adatok referencia elsődleges kulcs használatával így nem szükséges megkettőzni, az alábbi ábrán látható módon. Az eredeti adatok egy ténytábla nevezik.

![3. ábra - adatok minden index tábla által hivatkozott](./_images/index-table-figure-3.png)


Ez a módszer helyet takarít meg, és csökkenti a duplikált adatok kezelése. A hátránya, hogy rendelkezik-e egy alkalmazás egy másodlagos kulcs használatával adatok kereséséhez két keresési műveletek végrehajtásához. Keresse meg az elsődleges kulcs az adatok az index táblázat, és az elsődleges kulcs segítségével megkeresheti az adatokat a ténytábla rendelkezik.

A harmadik stratégia, ha a részlegesen normalizált index táblák, különböző kulccsal, hogy a gyakran lekért mezők szerint vannak rendezve. A ténytábla ritkábban hozzáférés mezőkre hivatkozhatnak. A következő ábra azt mutatja be milyen gyakran használt adatokhoz minden index táblázat ismétlődik.

![4. ábra - gyakran elért adatok ismétlődik minden index tábla](./_images/index-table-figure-4.png)


Az ezt a stratégiát akkor is egyensúlyt biztosítanak az első két megközelítés között. Az általános lekérdezések lehet adatokat beolvasni gyorsan egy egyetlen keresési használatával, amíg a lemezterület és a karbantartás többletterhelés nincs olyan jelentős, mint a teljes adatkészlet duplikálásakor.

Ha egy alkalmazás gyakran adatokat (például "Található összes ügyfél számára, hogy a Redmond, és amelyek a vezetékneve Smith") értékek kombinációja megadásával, sikerült megvalósítása a kulcsokat az elemeket az index táblázatban, a város összefűzése attribútum és a Vezetéknév attribútum. Az alábbi ábrán látható egy index táblázat összetett kulcsok alapján. A kulcsok az rekordokat, amelyek a város tartozhat azonos érték rendezi a város, majd Vezetéknév szerint.

![5. ábra – egy index táblázat összetett kulcsok alapján](./_images/index-table-figure-5.png)


Index táblák felgyorsíthatja a lekérdezési műveletek felett horizontálisan skálázott adatok, és különösen hasznosak, ahol a shard kulcs kivonat készül. A következő ábra azt szemlélteti, ahol a shard kulcsa kivonatát, az ügyfél-azonosító. Az index táblázat adatok rendezése nonhashed értéke (a város és a Vezetéknév), és adja meg a keresési Data a kivonatolt shard kulcsát. Ezt az alkalmazást ismételten kiszámítása a kivonat-kulcsok (drága művelet), ha egy tartományba eső adatok beolvasásához szükséges is mentheti, vagy adatlehívás nonhashed kulcs sorrendben kell. Például lekérdezés például a "Található összes ügyfél számára, hogy Redmond" gyorsan megoldhatók a egyező elemek megkeresése az index táblázatban, ahol minden fontosságúak tárolt összefüggő blokkban. Ezután kövesse az ügyféladatokat, a index tábla tárolja a shard kulcsokkal hivatkozik.

![6. ábra – egy index táblázat gyors keresési horizontálisan skálázott adatok megadása](./_images/index-table-figure-6.png)


## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- A terhelés, másodlagos indexek fenntartásának jelentős lehet. Kell elemezni és megérteni, hogy az alkalmazás által lekérdezéseket. Index táblák csak létrehozása, ha azok rendszeresen felhasználásra. Támogatja az alkalmazás nem hajtja végre, vagy hajt végre csak esetenként spekulatív index táblák ne hozzon létre.
- Az index táblázat adatainak duplikálása adhat hozzá jelentős terhelés tárolási költségek és az adatok többszörös lemásolását, karbantartásához szükséges beavatkozást.
- Egy alkalmazás adatok kereséséhez két keresési műveletek végrehajtásához, az eredeti adatok hivatkozó normalizált struktúra egy index táblázat végrehajtási szükséges. Az első művelet index tábla elsődleges kulcsa keres, és a második elsődleges kulcsot használja az adatok beolvasása.
- Ha a rendszer a szerződés magában foglalja egy index táblák számát keresztül nagyon nagy méretű adatkészletekhez, biztosítja az egységességet index táblák és az eredeti adatok közötti nehézkes lehet. A végleges konzisztencia modell körül alkalmazás is lehet. Insert, update vagy törli az adatokat, például egy alkalmazás képes annak a várólistára üzenetet és lehetővé teszik egy külön feladat végrehajtani a műveletet, és aszinkron módon referenciaadatok index táblákhoz karbantartása. A végleges konzisztencia kapcsolatos további információkért lásd: a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).

   >  A Microsoft Azure storage-táblákat tranzakciós frissítések támogatása (néven entitás csoport tranzakciók) azonos partíciójában végzett módosításokat. Ha egyazon partícióra kerüljenek a egy ténytábla és egy vagy több index táblák adatait tárolhatja, ez a szolgáltatás segítségével konzisztencia érdekében.

- Index táblák maguk lehet particionált vagy szilánkos.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta segítségével javíthatja a lekérdezések teljesítményét, ha egy alkalmazás gyakran kell kulccsal kívül az elsődleges (vagy a shard) kulcs adatainak beolvasása.

Ebben a mintában előfordulhat, hogy nem lehet hasznos:

- A adata "volatile". Egy index táblázat válhat elavult nagyon gyorsan hatástalanná teszik vagy a fenntartásának az index táblázat nagyobb, mint bármely megtakarított azt a terhelést.
- A kiválasztott másodlagos kulcs egy index táblázat mező nondiscriminating és egyszerre csak egy kis készletét értékek (például nemét).
- Egy másodlagos kulcs egy index táblázat kiválasztott mező alapján az adatértékek egyenlege magas ferde vannak. Ha a rekordokat a 90 %-a mezője azonos értéket tartalmaz, majd létrehozása, és ez a mező alapján adatokat kereshet egy index táblázatot karbantartó előfordulhat, hogy hozzon létre például további terhet jelentenek, mint egymás után keresztül az adatok vizsgálatát. Azonban ha lekérdezések nagyon gyakran célértéket, melyek a fennmaradó 10 %-ban, ez az index hasznos lehet. Ismerje meg, hogy működik-e az alkalmazás, és milyen gyakran által végrehajtott lekérdezéseket.

## <a name="example"></a>Példa

Az Azure storage táblázatokban egy kiválóan méretezhető kulcs/érték adattár a felhőben futó alkalmazások. Alkalmazások tárolásához, és az adatértékek lekéréséhez kulcs megadásával. Az adatok értékek tartalmazhatnak több mező, de adatelemet szerkezete fedett, a table storage, amely egyszerűen végzi, bájttömb adatelemet.

Az Azure storage-táblákat horizontális is támogatja. A horizontális két elemet, egy partíciót és sor kulcsot tartalmaz. Ugyanazzal a partíciókulccsal rendelkező elemek (szilánkok) tartalmazó partícióra vannak tárolva, és az elemek sor kulcs sorrendje a shard belül vannak tárolva. A TABLE storage a partíción belül sor kulcsértékei összefüggő számos alá tartozó adatlehívás lekérdezések végrehajtása van optimalizálva. Ha felhőalapú alkalmazásokhoz, Azure-táblákban információt tároló most felépítése, ez a szolgáltatás szem előtt az adatok szerkezetét.

Vegye figyelembe például olyan alkalmazás, amely filmek kapcsolatos információkat tárolja. Az alkalmazás gyakran lekérdezi filmek által genre (a művelet, dokumentált, korábbi, comedy, tragédiát, és így tovább). Egy Azure-tábla segítségével létrehozhat minden genre partíciók használatával a genre partíciókulcsnak, és a sorkulcs, mint a movie név megadásával a következő ábrán látható módon.

![7. ábra - Movie adataihoz az Azure tábla](./_images/index-table-figure-7.png)


Erre akkor kevésbé hatékony, ha az alkalmazást is kell lekérdezés filmek szereplő Főszerepben által. Ebben az esetben az Azure táblájába, amely különbséglemezként funkcionál egy index táblázat is létrehozhat. A partíciós kulcs a szereplő, és a sorkulcs movie nevét. Minden egyes szereplő adatok külön partíciók tárolódnak. Ha film csillaggal egynél több szereplő, az azonos movie több partíciót történik.

Mindegyik partíció tartja az első módszer a megoldást a fenti szakaszban leírt elfogadásával értékek movie adatainak másolhatja. Azonban valószínű, hogy minden movie többször replikálja (egyszer az egyes aktor), ezért előfordulhat, hogy hatékonyabb, ha részben denormalize az adatokat (például a más szereplője nevének) a leggyakoribb lekérdezések támogatja, és lehetővé teszik az alkalmazások beolvasása fennmaradó adatokat a meg nem találja a teljes körű információkat a genre partíciókat a partíciókulcs-ot. Ez a megközelítés a harmadik lehetőség a megoldás szakaszban írja le. A következő ábrán látható ezt a módszert használja.

![8. ábra - index táblák movie adatok működött szereplő partíciók](./_images/index-table-figure-8.png)


## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). Az index táblázat fenn kell tartani a Data azt indexeli a módosításokat. A felhőben nem lehet vagy nem megfelelő index frissítése ugyanabban a tranzakcióban, amely módosítja az adatok részeként műveleteket. Ebben az esetben egy idővel konzisztenssé megoldás, megfelelő. A végleges konzisztencia körülvevő problémákat ismerteti.
- [Horizontális mintát](https://msdn.microsoft.com/library/dn589797.aspx). Az Index táblázat mintát gyakran együtt használatos szilánkok használ adatokkal. A horizontális mintát adattárat felosztani szilánkok készlete módjáról nyújt részletesebb információt.
- [Materializált nézet mintát](materialized-view.md). Helyett az indexelési adatok támogatja az adatokat, akkor célszerű több adat materializált nézet létrehozásához. Ismerteti, hogyan támogatja az hatékony összefoglaló adatok előfeltöltött nézetek létrehozásával.
