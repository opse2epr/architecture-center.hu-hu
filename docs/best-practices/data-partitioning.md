---
title: "Adatparticionálási útmutató"
description: "Útmutató a szétválasztásának kezelhető és külön-külön elérhető partíciókat."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: c139fd1ef59ea94235cd9519dd064d0722cee3c9
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="data-partitioning"></a>Adatparticionálás
[!INCLUDE [header](../_includes/header.md)]

A sok nagy méretű megoldások adatok, kezelhető és érhető el külön külön partíciókra oszlik. A jó particionálási stratégia a következő előnyöket maximalizálása ugyanakkor minimalizálja a negatív hatások gondosan kell kiválasztani. Particionálás segíthet a méretezhetőség javítása, csökkentse a versengés és teljesítményének optimalizálásához. Egy másik particionálás előnye, hogy egy olyan mechanizmus adatok elosztjuk használatának nyújthat. Például úgy archiválhatók régebbi, kevésbé aktív (ritkán használt) adatok olcsóbb adattárolásra.

## <a name="why-partition-data"></a>Miért partícióazonosító adatokat?
A legtöbb felhőalapú alkalmazások és szolgáltatások tárolja, és a műveletek részeként adatainak beolvasása. Az adattároló, amely egy alkalmazás tervét a teljesítményt, adatátvitelt és méretezhetőséget biztosít a rendszer jelentős hatással lehet. A nagy méretű rendszerekhez gyakran alkalmazott egyik módszer-az adatok felosztása külön partíciókra.

> Ebben a cikkben kifejezés *particionálás* azt jelenti, hogy fizikailag felosztásával adatok be különálló adattároló folyamatán. Nincs ugyanaz, mint az SQL Server táblaparticionálást.

Adatok particionálása kínálhat több előnnyel jár. Például is alkalmazható annak érdekében, hogy:

* **Méretezhetőség javítása**. Ha egy önálló adatbázis rendszer növelheti, azt idővel eléri a fizikai hardver korlátozni. Ha több partíciót, amelyek mindegyike külön kiszolgáló üzemelteti, között megoszthatjuk az adatokat ki lehet terjeszteni a rendszer szinte határozatlan ideig.
* **A teljesítmény javítása**. Minden egyes partícióra adatelérési műveletek történjen az adatok kisebb kötetet. Feltéve, hogy az adatok particionálása megfelelő módon, particionálás is hatékonyabbá teszi a rendszer. Egynél több partíció-műveletek párhuzamosan futtatható. Mindegyik partíció található a hálózati késés csökkentése érdekében érdemes használó alkalmazások közelében.
* **Rendelkezésre állás fejlesztése a**. Az adatok elkülönítését a több kiszolgálóra Ezzel elkerülheti a hibaérzékeny pontok kialakulását. Ha egy kiszolgáló működése leáll, vagy éppen karbantartják tervezett, csak azokat az adatokat abban a partíció nem érhető el. A többi partíció műveletek továbbra is. A partíciók számának növelésével csökkenthető az egyetlen kiszolgáló hibát relatív hatása nem érhető el adat csökkentésével. Mindegyik partíció replikálása is csökkenthető a érintő műveletek egypartíciós hiba esélyét. Is lehetővé teszi, hogy külön kritikus fontosságú adatok, amelyek folyamatosan kell lennie, és magas rendelkezésre állású az alacsony érték, amely rendelkezik alacsonyabb rendelkezésre állási követelmények (például a rendszernapló fájljaiban).
* **A webalkalmazások biztonságának fokozására**. Az adatok és hogyan van particionálva természetétől függően esetleg a bizalmas és nem érzékeny adatok különböző külön partíciót, és ezért különböző kiszolgálókra vagy adatokat tárolja. Biztonsági majd kifejezetten optimalizálva a bizalmas adatok.
* **Működési rugalmasságot biztosítanak**. Sok lehetőségek finom műveletek hangolása, maximalizálva felügyelet hatékonysága és a költségek minimalizálása particionálás kínál. Például megadhatja különböző stratégiák felügyeletének, megfigyelésének, biztonsági mentés és visszaállítás és más felügyeleti feladatokra mindegyik partíció adatainak fontossága alapján.
* **Az adattároló használatának egyezés**. Particionálás lehetővé teszi, hogy mindegyik partíció adattár, a költségek és a beépített szolgáltatásokat, hogy az adattárolási ajánlatok alapján más típusú telepít. Például nagy bináris adatokat tárolhatja a blob-tárolóban, amíg a strukturált adatok több lehessen vonni a dokumentum-adatbázisban. További információkért lásd: [polyglot megoldás kialakításának] minták és gyakorlatok útmutató és [adatelérési kiválóan méretezhető megoldások: SQL használatával, nosql-alapú és polyglot adatmegőrzési] meg a A Microsoft webhelyén.

Egyes rendszerek nem valósítja meg particionálásból, mert akkor tekinthető előnyös, hanem egy költség. A profilkategóriák gyakori okai:

* Sok adattároló rendszerek nem támogatja az olyan illesztéseket partíciók között, és karbantartása a hivatkozási integritás egy particionált rendszerben nehéz lehet. Gyakran illesztések végrehajtásához szükséges integritási ellenőrzések és alkalmazáskód (a particionálási réteg), a kiegészítő i/o és alkalmazás bonyolultságát ennek eredményeként.
* Karbantartása partíciókra nincs mindig egy jelentéktelen feladat. A rendszer, ahol az adatokat az "volatile" szükség lehet egyensúlyba partíciók rendszeres időközönként a versengés és interaktív területek csökkentése érdekében.
* Néhány gyakori eszközök nem működnek természetes particionált adatokkal.

## <a name="designing-partitions"></a>Partíciók tervezése
Adatok különböző módokon lehet particionálni: vízszintesen, függőlegesen vagy funkciók. Stratégia attól függ, hogy az adatokat, és az alkalmazások és szolgáltatások által használt adatok követelményeinek particionálás okát.

> [!NOTE]
> Ez az útmutató ismerteti a particionálási sémák oly módon, amely független a mögöttes adatokat tároló technológia magyarázatát olvashatja. A számos különböző típusú adatokról, beleértve a relációs és NoSQL-adatbázisok alkalmazhatók.
>
>

### <a name="partitioning-strategies"></a>Particionálási stratégia
Az adatok particionálása három tipikus stratégia a következő:

* **Vízszintes particionálás** (gyakran nevezik *horizontális*). Ezt a stratégiát mindegyik partíció önálló adattárat, de az összes partíció rendelkezik ugyanazon séma. Mindegyik partíció is ismert, egy *shard* , valamint az adatok, például az ügyfelek e-kereskedelmi alkalmazás meghatározott készletének a rendeléseket megadott részhalmazára.
* **A vertikális particionálás**. Ezt a stratégiát mindegyik partíció rendelkezik elemek mezők az adattárban. A mezők használati a minta alapján vannak felosztva. Gyakran használt mezőket kell elhelyezni, például egy függőleges partícióban, és kevesebb mint egy másik gyakran használt mezőket.
* **Funkcionális particionálás**. Ezt a stratégiát, az adatok szerint felhasználásukról minden kötött környezet által a rendszer összesített értéket. Például az elektronikus kereskedelmi rendszer, hogy megvalósítja külön üzleti funkciók számlázásra, és Termékleltár kezelése számla adatokat tárolhatnak egy partíció és a termék leltáradatok egy másik.

Fontos megjegyezni, hogy az itt leírt három stratégiák egyesíthetők. Nem találhatók kölcsönösen kizárja egymást, és azt javasoljuk, hogy érdemes azokat az összes particionálási sémát tervezésekor. Például előfordulhat, hogy adatokat felosztani szilánkok, és majd a vertikális particionálást használnak a további megkönnyíti az egyes shard lévő adatokat. Ehhez hasonlóan az adatok egy funkcionális partíció szilánkok (amely szintén függőleges particionálhatónak) felosztható.

Minden egyes stratégia eltérő követelmények azonban egy ütköző problémák száma is módosíthatja. Értékelje ki és elosztása mindegyik egy particionálási sémát, amely megfelel az általános adatfeldolgozás teljesítménycéloknak, a rendszer tervezésekor kell. Az alábbi szakaszok részletesebben stratégiák mindegyikének vizsgálatát.

### <a name="horizontal-partitioning-sharding"></a>Vízszintes particionálás (horizontális)
1. ábra a vízszintes particionálás vagy horizontális áttekintését tartalmazza. Ebben a példában a termék leltáradatokat a termékkulcs alapján szilánkok van felosztva. Minden egyes shard shard kulcsok (A – G és H-Z), ábécésorrendben vannak rendszerezve összefüggő számos adatait tartalmazza.

![A partíciós kulcs alapján vízszintesen particionálási (horizontális) adatok](./images/data-partitioning/DataPartitioning01.png)

*1. ábra. A partíciós kulcs alapján vízszintesen particionálási (horizontális) adatok*

Horizontális segít a terhelés eloszlatva további számítógépek, ami csökkenti a versengés, és javítja a teljesítményt. A további kiszolgálók futó szilánkok további hozzáadásával, méretezhető, a rendszer.

A legfontosabb tényező, ha a particionálási stratégia megvalósítása a választott horizontális kulcs. A kulcs módosítása után a rendszer a művelet nehézkes lehet. A kulcs győződjön meg arról, hogy adatok particionálása, hogy a munkaterhelés, még akkor is lehetséges a szilánkok között.

Ne feledje, hogy különböző szilánkok nem hasonló adatmennyiséggel tartalmaz. Ehelyett a további fontos szempont kérések száma elosztása érdekében. Lehet, hogy néhány szilánkok nagyon nagy, de minden elem egy kevés hozzáférési művelet tárgyát. Előfordulhat, hogy más szilánkok kisebb, de egyes elemek gyakrabban érhető el. Fontos továbbá győződjön meg arról, hogy egyetlen shard nem haladja meg a skála korlátoknak (kapacitás és egyéb feldolgozási erőforrás) az adattár tárolására a shard használt.

Ha egy horizontális skálázási sémát használja, ne hozzon létre olyan csatlakozási pontokhoz (vagy a működés közbeni partíciók), amely hatással lehet a teljesítmény és rendelkezésre állás. Például helyett a felhasználó nevében első betűjének használatakor kivonatát, ügyfél-azonosító, hogy akadályozza meg, amelyek közös és ritkább kezdeti betűket való egyenetlen eloszlását. Ez a egy tipikus technika, amely segít az adatok több egyenletesen szét partíciókat.

Válasszon egy horizontális skálázási kulcsot, amely minimálisra csökkenti a nagy szilánkok ossza fel kisebb kódrészletek, minden jövőbeli követelmények nagyobb partíciókra kis szilánkok coalesce, vagy módosítsa a séma, amely leírja a partíciók készlete tárolt adatokat. Ezek a műveletek nagyon időigényes lehet, és előfordulhat, hogy egy vagy több szilánkok offline állapotba helyezése, hogy végrehajtása közben.

Ha a rendszer replikálja a szilánkok, esetleg megőrzése a replikák online amíg mások vágási, egyesített, vagy újra konfigurálni. Azonban a rendszer módosítania kell a újrakonfigurálását közben az alábbi szilánkok található adatok végrehajtható műveletek korlátozására. Például a replikák adatait, amely akkor jelentkezhet, amíg a szilánkok átalakult inconsistences korlátozhatja a csak olvasásra jelölhető.

> További részletes információt és útmutatást ezeket a szempontokat számos, és hasznos technikákat tervezése adatokat tárolja, hogy megvalósítása vízszintes particionálás, a következő témakörben: [horizontális mintát].
>
>

### <a name="vertical-partitioning"></a>A vertikális particionálás
A tevékenység leggyakoribb felhasználási függőleges particionálásból, hogy csökkentse az i/o és beolvasása az elemek, társított teljesítmény költségek a leggyakrabban elért. 2. ábrán látható egy példa a vertikális particionálás. Ebben a példában minden egyes adatelem más tulajdonságokkal különböző partíciók vannak használatban. Egy partíció, amelyhez a gyakrabban, beleértve a nevét, leírását és termék adatait ára adatokat tartalmazza. Egy másik tartalmazza a kötet árfolyam- és az utolsó rendezett dátum.

![A minta használati függőleges particionálási adatforgalma](./images/data-partitioning/DataPartitioning02.png)

*2. ábra. A minta használati függőleges particionálási adatforgalma*

Ebben a példában az alkalmazás rendszeresen lekérdezi a termék nevét, leírását és ár az ügyfél számára a termék részletei megjelenítésekor. A készlet szinten, és ha a gyártó által rendezett az a termék utolsó dátuma tartják külön partícióra, mert a két elem általában együtt használhatók.

A particionálási sémát rendelkezik előnye, hogy a viszonylag lassú-áthelyezése adatokat (a termék nevét, leírását és ár) elkülönül a dinamikus adatokat (készlet szint és utolsó rendezett dátuma). Egy alkalmazás hasznosnak találja a hasznos gyorsítótárazza a lassan adatokat a memóriában, ha a gyakori hozzáférést igénylő azt.

Jellemző még például az a jó particionálási stratégia, hogy a legbiztonságosabbá bizalmas adatokat. Például ehhez hitelkártyaszámokat és a megfelelő biztonsági-ellenőrzési számok tárolása különálló partíció.

A vertikális particionálás egyidejű hozzáférés szükséges az adatok mennyisége is csökkentheti.

> A vertikális particionálás belül a tárolóban, részben normalizálása szeretné bontani a entitás entitás szinten működik egy *széles* egy elem *szűkítéséhez* elemek. Az kiválóan oszlop célú adatok tárolókhoz, például a HBase és Cassandra. Ha az adatok egy gyűjtemény az oszlopok nem valószínű, hogy módosítsa, is megpróbálhatja oszlop tárolók az SQL Server használatával.
>
>

### <a name="functional-partitioning"></a>Funkcionális particionálás
A rendszerek, ahol egy kötött környezet különböző üzleti terület vagy az alkalmazás szolgáltatások azonosítható működési particionálás biztosít technika elkülönítési és az adatok teljesítmény javításához. Egy másik közös működési particionálás használata csak olvasható adatok számára a jelentéskészítéshez használt írható-olvasható adatok elkülönítéséhez. 3. ábra működési particionálás áttekintése látható, ahol a leltáradatok ügyféladatok elkülönül.

![A kötött környezetére vagy nevű altartomány funkcionálisan particionálási adatforgalma](./images/data-partitioning/DataPartitioning03.png)

*3. ábra. A kötött környezetére vagy nevű altartomány funkcionálisan particionálási adatforgalma*

A jó particionálási stratégia csökkentheti az adatok hozzáférési versengés a rendszer különböző részei között.

## <a name="designing-partitions-for-scalability"></a>Partíciók méretezhetőségének tervezése
Elengedhetetlen méretét és a munkaterhelés minden partíció esetében fontolja meg, és elosztása őket, hogy az adatok maximális méretezhetőség eléréséhez terjesztése. Azonban meg kell is partícióazonosító az adatokat, hogy nem haladja meg a egypartíciós tárolók méretezési korlátok.

Partíciókat méretezhetőséget a tervezésekor, kövesse az alábbi lépéseket:

1. Elemezze az alkalmazást a hozzáférési minták, például a minden egyes lekérdezés által visszaadott eredményhalmaz megértéséhez, a hozzáférést, a kiszolgálókezelőtől késés és a kiszolgálóoldali gyakoriságát számítási ügyféloldali bővítmények feldolgozási követelményeivel. Sok esetben néhány főbb entitások a szükséges erőforrások a legtöbb fog igény.
2. Az elemzés segítségével határozhatja meg a jelenlegi és jövőbeli méretezhetőségi célok, például az adatok méretével és a munkaterhelés. A partíciókat méretezhetőséget cél teljesítéséhez majd el az adatokat. A vízszintes particionálási stratégia kiválasztása a megfelelő shard kulcs fontos győződjön meg arról, hogy a terjesztés még az. További információkért lásd: a [horizontális mintát].
3. Győződjön meg arról, hogy megfelelőek-e mindegyik partíció számára elérhető erőforrások kezeléséhez a méretezhetőségi követelményeinek adatok mérete és a teljesítmény tekintetében. Például a csomópont, amelyen egy partíció lehet, hogy korlátozzák rögzített tárhely, feldolgozás power, vagy a hálózati sávszélességet biztosít mennyiségét. Ha az adatok tárolási és feldolgozási követelményei valószínűleg meghaladja ezt a korlátot, pontosítsa a particionálási stratégia, vagy ossza fel a kimenő adatforgalmat tovább szükség lehet. Például egy méretezhetőség megközelítés lehet eltérő a naplózási adatokat, az alkalmazás alapszolgáltatások külön. Ez a teljes adattárolási követelmények az a csomópont a méretezési túllépő megelőzése érdekében különálló adattároló használatával teheti meg. Ha adattárolókhoz teljes száma túllépi ezt a csomópontot, külön tárolócsomópontok használandó szükség lehet.
4. Segítségével győződjön meg arról, hogy a várt módon terjeszt-e az adatok, és, hogy a partíciók kezelni tud-e be, hogy ki őket a rendszer figyelni. Akkor lehet, hogy a nem egyezik az elemzés várható használat. Ebben az esetben esetleg a partíciók egyensúlyba. Ennek hiányában szükség lehet ahhoz, hogy a szükséges egyenleg a rendszer bizonyos részeihez átírása.

Vegye figyelembe, hogy néhány felhőkörnyezetekben erőforrásokat infrastruktúra határok tekintetében. Győződjön meg arról, hogy a kiválasztott határ határain biztosít-e az adatok tárolására, a feldolgozási teljesítmény és a sávszélesség mennyiségét várható növekedésének elegendő helyet.

Például az Azure table storage használatakor egy foglalt shard további erőforrások, mint amennyi rendelkezésre áll a tanúsítványigénylések egyetlen partícióra lehet szükség. (Nincs a megadott korlát a kötet, hogy egy adott időn belül egyetlen partícióra által is kezelt kérelem. A lap [az Azure storage méretezhetőségi és Teljesítménycélok] a Microsoft webhelyén olvashat.)

 Ha ez a helyzet, a shard módosítania kell particionálni a terhelés terjednek. Ha a teljes méret vagy a sebességét, ezek a táblázatok meghaladja a kapacitását, egy tárfiókot, további tárhely-fiókok létrehozásához, és ezek a fiókok a táblák elosztva szükség lehet. Ha a storage-fiókok száma meghaladja az előfizetés rendelkezésre álló fiókok számát, majd szükség lehet több előfizetéssel használandó.

## <a name="designing-partitions-for-query-performance"></a>A lekérdezési teljesítmény partíciók tervezése
Gyakran is súlyozott lekérdezési teljesítményt, kisebb adatkészletek használatával, és futtassa a párhuzamos lekérdezések. Mindegyik partíció kell tartalmaznia a teljes adatkészlet kis részét. A kötet csökkenése javíthatja a lekérdezések teljesítményét. Particionálás azonban nem tervezéséhez és megfelelő konfigurálása egy adatbázis helyett. Például győződjön meg arról, hogy rendelkezik a szükséges indexek helyen egy relációs adatbázis használata.

A lekérdezési teljesítmény partíciók tervezésekor, kövesse az alábbi lépéseket:

1. Vizsgálja meg az alkalmazás követelményeket és a teljesítmény:
   * Az üzleti követelmények segítségével határozhatja meg a kritikus lekérdezések, amelyek mindig gyorsan kell elvégezni.
   * A rendszer lassan végző lekérdezéseket azonosításához figyelése.
   * Hoz létre, mely lekérdezéseket gyakran kerül sor. Előfordulhat, hogy minden egyes lekérdezés egyetlen példányát minimális költség, de lehet, hogy az erőforrások összesített felhasználása jelentős. Előfordulhat, hogy a különböző partíció, vagy akár egy gyorsítótár lekérdezések által beolvasott adatok elválasztásához előnyös.
2. Az adatok, amelyek teljesítménycsökkenést okoz. partíció:
   * Mindegyik partíció méretének korlátozására, hogy a lekérdezési válaszidőt a cél.
   * A szilánkok kulcs tervezze meg, hogy az alkalmazás könnyedén megtalálhatja a partíció Ha webkiszolgálókból vízszintes particionálás. Ez megakadályozza, hogy a lekérdezés nem kell minden partíció keresztül vizsgálata.
   * Érdemes lehet a partíció helyét. Ha lehetséges próbálja meg, amely földrajzilag megközelítik az alkalmazások és az azt elérő felhasználók partíciók adatok tartása.
3. Ha egy entitás átviteli sebesség és a lekérdezési teljesítmény követelmények, használja a működési particionálás, hogy az entitás alapján. Ha ez nem megfelel, alkalmazza a vízszintes particionálás is. A legtöbb esetben elegendő egyetlen jó particionálási stratégia, de egyes esetekben hatékonyabb, ha mindkét stratégia kombinálni.
4. Érdemes lehet aszinkron teljesítmény javítása érdekében partíció párhuzamosan futó lekérdezések.

## <a name="designing-partitions-for-availability"></a>A rendelkezésre állás érdekében partíciók tervezése
Adatok particionálása javíthatja az alkalmazások rendelkezésre állását biztosítja, hogy a teljes adatkészlet nem jelent-e a hibaérzékeny pontok kialakulását, és, hogy az adatkészlet egyes részhalmaza függetlenül is kezelhetők. Kritikus fontosságú adatok partíciók replikálása szintén javíthatják a rendelkezésre állási.

Tervezése és megvalósítása partíciók, vegye figyelembe a következő rendelkezésre állási befolyásoló tényezők:

* **Hogyan kritikus az adatokat, hogy az üzleti tevékenységre**. Egyes adatok közé tartozik a kritikus üzleti adatok, például banki tranzakciók vagy számlázási adatokat. Egyéb adatok közé tartozik a kevésbé fontos működési adatokat, például a naplófájlok, teljesítmény nyomkövetési adatokat, és így tovább. Különböző típusú adatok azonosítása, után vegye figyelembe:
  * Kritikus fontosságú adatok tárolása egy megfelelő biztonsági mentési terv magas rendelkezésre állású partíciók.
  * Létrehozó különálló felügyeleti és figyelési mechanizmusok vagy a különböző criticalities adatkészlet egyes eljárások. Helyezze olyan adatok, amelyek a partícióra kritikusságúként azonos szinten, hogy egy megfelelő gyakorisággal, biztonsági másolat együtt. Például kívánt banki tranzakciók adatainak tárolásához módosítania kell biztonsági mentését gyakrabban kívánt naplózás vagy a nyomkövetési adatok tárolására.
* **Hogyan kezelheti az egyes partíciók**. Partíciók támogatására független felügyeleti és karbantartási tervezése több előnyökkel jár. Példa:
  * A partíció nem sikerül, ha akkor állíthatók helyre egymástól függetlenül nem befolyásolja a többi partíció adatainak elérhető alkalmazásokat példányai.
  * Particionálás adatokat földrajzi területenként lehetővé teszi, hogy minden hely csúcsidején elvégzi az ütemezett karbantartási feladatok. Győződjön meg arról, hogy partíciók nem túl nagy, ebben az időszakban nem lehet végrehajtani a tervezett karbantartások megelőzése érdekében.
* **E kritikus fontosságú adatok partíciók közötti replikáció**. Ezt a stratégiát javíthatja rendelkezésre állásáról és teljesítményéről, de azt is vezethet konzisztenciabeli problémákat. Egy partíció adatait szinkronizálni kell minden replika végzett módosítások időt vesz igénybe. Ebben az időszakban a különböző partíciók különböző értékeket fogja tartalmazni.

## <a name="understanding-how-partitioning-affects-design-and-development"></a>Hogyan particionálás befolyásolja a tervezési és fejlesztési ismertetése
A tervezési és a rendszer fejlesztési particionálás használatával hozzáadja összetettségét. Fontolja meg a particionálás a alapvető részeként a rendszer kialakítása még akkor is, ha a rendszer először csak olyan egyetlen partícióra. Meg oldani a particionálás utólag, mint amikor a rendszer megkezdi a teljesítmény- és méretezhetőségi problémákkal küzdenek, ha összetettségét növeli a rendszer, mivel már van egy működő rendszer fenntartásához.

Ha frissíti a rendszer átfogó particionálás ebben a környezetben, azt szükségessé teszi az adatok hozzáférési logikai módosítása. Azt is magába foglaló áttelepítése szét, partíciók gyakran közben felhasználók kellene továbbra is használja a rendszer a meglévő adatok nagy mennyiségben.

Bizonyos esetekben particionálás nem fontos, mert a kezdeti adatkészlet kicsi, és egy adott kiszolgáló könnyen kezelhető. Erre akkor lehet igaz, a rendszer a kezdeti méretét felüli nem várt, de számos kereskedelmi rendszer kell bontsa ki a felhasználók növekszik számának. A bővítés általában csatolni egy növekedésének adatok mennyiségét.

Is fontos tisztában lenni azzal, hogy particionálás nem mindig nagy adattárolókhoz függvényében. Például egy kis adattár fokozottan érheti el a több száz egyidejű ügyfelek. Ebben az esetben az adatok particionálása segíthet csökkentheti a versengés és javíthatja a teljesítményt.

Adatok particionálási sémát tervezésekor, vegye figyelembe a következő szempontokat:

* **Ahol lehetséges, a leggyakrabban használt adatbázis-műveletek adatok együtt tartása a kereszt-partíció adatelérési műveletek minimalizálása érdekében minden partíció**. Több időt vesz igénybe, mint csak belül egyetlen partícióra lekérdezése közötti partíciók lekérdezése lehet, de a partíciókat a lekérdezések egyetlen halmazát optimalizálása kedvezőtlen hatással lehet más részhalmazához rendelésére lekérdezések. Partíciók között lekérdezése nem kerülheti el, ha minimálisra csökkenthető a lekérdezés idejét párhuzamos lekérdezések futtatásával, és az alkalmazáson belül az eredmények összesítése. Előfordulhat, hogy ez a módszer nem egyes esetekben, amikor szükséges ahhoz, hogy egy lekérdezés eredményt beszerezni, használhatja a következő lekérdezés nem lehetséges.
* **Ha lekérdezések akkor viszonylag statikus referenciaadatok, például az irányítószámot táblák vagy termék listák használatát, javasoljuk, hogy ezeket az adatokat az összes különböző partíciók külön keresési művelet kapcsolatos követelmények csökkentése érdekében a partíciók replikálása**. Ez a megközelítés is csökkentheti a referenciaadatok válik, hogy a nagy forgalom között a teljes rendszer "Forró" dataset valószínűségét. Van azonban a referenciaadatok előforduló módosítások szinkronizálása társított további költségek.
* **Ahol lehetséges, minimálisra csökkenthető a hivatkozási integritás követelményei közötti függőleges és funkcionális partíciók**. Ezek a rendszerek a magának az alkalmazásnak karbantartásáért felelős a hivatkozási integritás partíciók között, amikor az adatok frissítése és felhasznált. Lekérdezés kell adatok között több partíció, amelyhez csatlakozni adatok csak a partíción belül, mert az alkalmazás általában egymást követő lekérdezések végrehajtásához, egy kulcs, és egy külső kulcs lekérdezések mint lassabban működik. Ehelyett érdemes végez replikálást vagy vonja normalizálása a vonatkozó adatokat. A lekérdezés idejét, amennyiben szükségesek a kereszt-partíció illesztések minimalizálása érdekében párhuzamos lekérdezések futtatása során a partíciók, és az adatokat az alkalmazásban.
* **Vegye figyelembe, hogy milyen hatással a particionálási sémát előfordulhat, hogy az adatok konzisztenciájának a partíciók között.** Mérlegelje, hogy az erős konzisztencia feltétele ténylegesen. Ehelyett egy általánosan használt megközelítés a felhőben, hogy alkalmazza a végleges konzisztencia. Mindegyik partíció adatait a külön-külön frissül, és az alkalmazáslogikát biztosítja, hogy a frissítések összes sikeresen megadta. Az adatok lekérdezése egy idővel konzisztenssé művelet futtatása közben felmerülő inkonzisztenciák is kezeli. A végleges konzisztencia kapcsolatos további információkért lásd: a [adatok konzisztencia ismertetése].
* **Vegye figyelembe, hogy lekérdezések hogyan keresse meg a megfelelő partíció**. Lekérdezés partíciók keresse meg a szükséges adatokat kell beolvasni, ha nincs jelentős hatással a teljesítményre, még akkor is, ha több párhuzamos lekérdezések szolgáltatások futnak. A függőleges és funkcionális particionálási stratégiák használó lekérdezések természetes adhatja meg a partíciók. Azonban vízszintes particionálás (horizontális) teheti nehéz elemének megkeresése, mert minden shard ugyanazon séma. Egy tipikus horizontális megoldás, hogy karbantartása, hogy segítségével megkeresheti az adatok elemeket shard helyét. Ez a térkép az alkalmazás horizontális logikája megvalósított, vagy ha támogatja az áttetsző horizontális tartja fenn a tárolót.
* **Vízszintes jó particionálási stratégia használata esetén vegye figyelembe a szilánkok rendszeresen újraelosztás**. Ezzel a megoldással mérete és a munkaterhelés minimalizálása érdekében a csatlakozási pontokhoz, lekérdezési teljesítmény maximalizálása és fizikai tárhelyet korlátozások kerülő egyenletes elosztása az adatokat. Ez azonban egy összetett feladat, amelyek gyakran egy egyéni eszköz vagy folyamat használatát igényli.
* **Mindegyik partíció replikálja, ha meghibásodása elleni további védelmet nyújt**. Ha nem sikerül egy replikát, lekérdezések irányítható a működő példány felé.
* **Ha eléri a jó particionálási stratégia fizikai korlátai, szükség lehet egy másik szintre méretezhetőségét kiterjeszteni**. Például ha az adatbázis szintjén a particionálás, szükség lehet keresse meg, vagy a partíciók több adatbázis replikálása. Particionálás már van az adatbázis szintjén, és fizikai korlátozások problémává, azt jelentheti, hogy szeretné-e keresse meg, vagy a partíciók több birtokosi fiókok replikálása.
* **Több partíció adatait hozzáférő tranzakciók elkerülése**. Néhány tárolók megvalósítása tranzakciós adatkonzisztencia és műveletek integritását, amely adatokat, de csak abban az esetben, ha az adatok találhatók egyetlen partícióra módosítása. Ha több partíciót keresztül kell dokumentumos tranzakciótámogatást, valószínűleg akkor valósítható meg ez az alkalmazás logikája, mert a legtöbb particionálási rendszerek nem tartalmazzák a natív támogatást.

Minden adattárolókhoz szükséges néhány működési kezelése és figyelése. A feladatok között lehet adatok betöltése, biztonsági mentése és adatok helyreállításához, adatok átrendezése és győződjön meg arról, hogy a rendszer megfelelően és hatékonyan működik-e.

Vegye figyelembe a következő tényezőket, amelyek hatással vannak az operatív felügyeleti:

* **Megfelelő felügyeleti és működési feladatok megvalósításához, amikor az adatok particionálása**. Ezek a feladatok állhatnak biztonsági mentése és visszaállítása, adatok, a rendszer, és más felügyeleti feladatok figyelése archiválása. Például a biztonsági mentési és visszaállítási műveletek során karbantartása logikai konzisztencia kihívást lehet.
* **Az adatok betöltése az több partíciót és az új adatok, amelyek más forrásból érkező van**. Egyes eszközök és segédeszközök nem támogathatja horizontálisan skálázott adatok műveletek, például az adatok betöltését a megfelelő partíció. Ez azt jelenti, hogy előfordulhat, hogy kell létrehozni, vagy szerezzen be új eszközök és segédprogramok.
* **Archivált, és törli az adatokat rendszeres időközönként**. Partíciók túlzott növekedést megelőzése érdekében szeretné archiválni, és törli az adatokat rendszeres időközönként (például havonta). Egy másik archiválási séma megfelelő adatok átalakítására szükség lehet.
* **Hogyan keresse meg az adatok épségével kapcsolatos problémák**. Fontolja meg egy olyan partíciót, egy másik objektuminformáció hivatkozó bármely adatok épségével kapcsolatos problémák, például az adatok kereséséhez rendszeres folyamatának futtatása. A folyamat vagy javítsa ki ezeket a problémákat automatikusan vagy manuálisan a problémák megoldására operátor riasztás tett kísérlet lehet. Például egy e-kereskedelmi alkalmazás rendelésinformációkat tárolható előfordulhat, hogy egy partíció, de a sor elemek minden rendelés alkotó egy másik lehet tartani. Adatok hozzáadása a többi partíció kell egy rendelés folyamatán. Ha ez a folyamat sikertelen, hiba előfordulhat, hogy lehet sor tárolt elemeket, amely nincs megfelelő rendelés.

Többféle tárolási technológiákat általában rendelkeznek a saját funkciók támogatják a particionálást. A következő szakaszok összegzik a beállításokat, amelyeket a rendszer által az Azure-alkalmazások gyakran használt adatok tárolókon. Azt is leírják, alkalmazásokat, amelyek leginkább kihasználhatják ezeket a szolgáltatásokat tervezéséhez.

## <a name="partitioning-strategies-for-azure-sql-database"></a>Az Azure SQL Database particionálási stratégia
Az Azure SQL-adatbázis egy relációs adatbázis-a-szolgáltatás, amely a felhőben. Microsoft SQL Server alapul. A relációs osztja a táblázatok adatait, és minden tábla tartalmazza a sorokat sorozataként entitások kapcsolatos információkat. Minden egyes sor tartalmazza az adatokat egy entitás egyedi mezők oszlopok. A lap [Mi az Azure SQL Database?] a Microsoft webhely létrehozása és használata az SQL-adatbázisok vonatkozó részletes dokumentációt nyújt.

## <a name="horizontal-partitioning-with-elastic-database"></a>Vízszintes particionálás rugalmas adatbáziskészlettel
Egy SQL-adatbázis a megadott korlát a kötet tartalmazhat adatot tartalmaz. Átviteli sebesség architekturális tényezők és az azt támogató létesített egyidejű kapcsolatok számát korlátozza. A rugalmas adatbázis-szolgáltatás SQL-adatbázis támogatja az SQL-adatbázis horizontális skálázást. Rugalmas adatbázist használja, a szilánkok, amely több SQL-adatbázisok vannak elosztva az adatok is partícióazonosító. Hozzáadhat, vagy távolítsa el a szilánkok, kezelni kívánt adatok mennyisége növekszik és csökken. Rugalmas adatbázis használatával is csökkenthető a versengés terhelését azáltal, hogy az adatbázisok közötti.

> [!NOTE]
> A rugalmas adatbázis megtalálható az összevonási szolgáltatás számára az Azure SQL adatbázis. Meglévő SQL-adatbázis összevonási telepítések áttelepíthetők rugalmas adatbázis áttelepítési összevonási segédprogram segítségével. Azt is megteheti Ha a forgatókönyv nem alkalmasnak természetesen a rugalmas adatbázis által nyújtott szolgáltatások saját horizontális mechanizmus is létrehozható.
>
>

Minden egyes shard, SQL-adatbázis lett megvalósítva. A szilánkok tárolására képes több adatkészlet (a továbbiakban a *shardlet*). Az egyes adatbázisok fenntartja a benne található shardlets leíró metaadatok. Egy shardlet lehet egyetlen adat, vagy egy csoport shardlet azonos kulccsal rendelkező elemek. Például ha egy több-bérlős alkalmazás horizontális adatokat, a shardlet kulcs lehet a bérlő azonosítója, és minden adat egy adott bérlő lehessen vonni a azonos shardlet részeként. Más bérlők adatokat a különböző shardlets kell tartani.

Feladata a programozói rendelje hozzá a dataset shardlet kulccsal. Egy másik SQL-adatbázist egy globális shard térkép manager funkcionál. Ezt az adatbázist a rendszer a szilánkok és shardlets listáját tartalmazza. Egy ügyfélalkalmazás tárolt adatokkal először csatlakozik a globális shard térkép manager adatbázisát hozzájutni a shard térkép (szilánkok és shardlets felsoroló), amely, majd gyorsítótárba helyezi azt a helyileg.

Az alkalmazás ezután a megfelelő shard útvonal kérelmek ezt az információt használja. Ez a funkció egy API-k, amelyek szerepelnek az Azure SQL Database rugalmas adatbázis ügyféloldali kódtár, amely a NuGet-csomagként érhető sorozatát takarja. A lap [rugalmas adatbázis-szolgáltatások áttekintése] a Microsoft webhelyén biztosít a rugalmas adatbázis egy átfogóbb bemutatása.

> [!NOTE]
> A globális shard manager adatbázist késés csökkentésére, valamint a rendelkezésre állás fejlesztése a replikálhatja. Ha az adatbázis a Premium árképzési szinteket egyikének használatával valósítja meg, konfigurálhatja a aktív georeplikáció folyamatosan adatok különböző régiókban adatbázisok másolása. Az adatbázis másolatának létrehozása minden régióban, amelyben a felhasználók alapulnak. Ezután konfigurálja az alkalmazás ezen példányának beszerzéséhez a szilánkok térkép való kapcsolódáshoz.
>
> Egy másik módszert is, hogy Azure SQL adatszinkronizálás vagy egy Azure Data Factory-folyamat használja a shard manager adatbázist régiók közötti replikáció. Az űrlap replikációs rendszeres időközönként fut, és több megfelelő, ha a shard térkép ritkán módosul. Emellett a shard manager adatbázist nem kell a prémium tarifacsomag használatával hozható létre.
>
>

Rugalmas adatbázis biztosítja az adatok leképezése shardlets, és tárolja őket szilánkok két sémák:

* A **lista shard térkép** egy kulcs és egy shardlet közötti társítás ismerteti. Például egy több-bérlős rendszerben az egyes bérlők számára az adatok is kell tartozó egyedi kulcs és a saját shardlet tárolja. Garantálja az adatvédelmi és elkülönítési (Ez azt jelenti, hogy egy bérlő megakadályozása kimerítsék érhető el, hogy mások számára az adatok tárolási erőforrások), minden shardlet saját shard megtartható.

![Egy lista shard térkép használatával bérlői adatokat külön szilánkok használatával](./images/data-partitioning/PointShardlet.png)

*4. ábra. Egy lista shard térkép használatával bérlői adatokat külön szilánkok használatával*

* A **tartomány shard térkép** összefüggő kulcs értékek és a shardlet közötti társítás ismerteti. A több-bérlős példa dedikált shardlets végrehajtási alternatívájaként korábban leírt csoportosíthatja az adatokat, azon belül az azonos shardlet (mindegyiket a saját kulcs) bérlők. Ez a séma olcsóbb az első (mivel a bérlők adatok tárolási erőforrások megosztása), de fennáll a kockázata, csökkentett adatvédelem és elkülönítési is létrehoz.

![Egy tartomány shard térkép használatával adatok köre bérlők által a shard használatával](./images/data-partitioning/RangeShardlet.png)

*5. ábra. Egy tartomány shard térkép használatával adatok köre bérlők által a shard használatával*

Vegye figyelembe, hogy egyetlen shard tartalmazhatnak több shardlets adatai. Például használhatja lista shardlets ugyanazt a shard különböző nem összefüggő bérlők esetén az adatok tárolásához. Bár ezek kiiktatása keresztül különböző maps a globális shard térkép manager adatbázisában tartomány shardlets és ugyanazt a shard, a lista shardlets is kombinálhatók. (A globális shard manager adatbázist tartalmazhat több shard maps.) 6. ábra mutatja be ezt a módszert használja.

![Több shard végrehajtási leképezhető](./images/data-partitioning/MultipleShardMaps.png)

*6. ábra. Több shard végrehajtási leképezhető*

A particionálási sémát, amely megvalósítása jelentős hatással lehet a rendszer teljesítményét. A sebesség, amellyel szilánkok kell hozzáadni vagy eltávolítani, vagy a sebesség, amellyel adatokat kell particionálni szilánkok között is érinti. A rugalmas adatbázis partíció adatokhoz való használatakor, vegye figyelembe a következő szempontokat:

* Adatok együtt használja ugyanazt a shard a csoportot, és használatban van a több szilánkok adatok elérését igénylő műveletek elkerülése érdekében. Ne feledje, hogy rugalmas adatbáziskészlettel egy shard önálló SQL-adatbázis, valamint az Azure SQL Database nem támogatja az adatbázisok közötti illesztések (amelyeket hajtható végre az ügyféloldali). Továbbá ne feledje, hogy az Azure SQL Database, a hivatkozási integritási megkötésekkel, eseményindítók és egy adatbázisban tárolt eljárások nem hivatkozhat egy másik objektum. Ezért ne kialakítási függőségek között szilánkok olyan rendszerre. SQL-adatbázis azonban tartalmazhatnak gyakran használják a lekérdezések és egyéb műveletekhez referenciaadatok példányait táblákhoz. Ezek a táblázatok nem rendelkeznek minden megadott shardlet tartozik. Ezek az adatok replikálása szilánkok segítségével távolítsa el a szükséges adatbázisokat kiterjedő illesztési adatok. Ideális esetben az ilyen adatok lehetnek statikus, illetve lassan lecsökkentheti a replikációs beavatkozást, és csökkenti a veszélyét annak, hogy azt elévültek.

  > [!NOTE]
  > Bár az SQL-adatbázis nem támogatja az adatbázisok közötti csatlakozik, a kereszt-shard lekérdezések rugalmas adatbázis API-val végezheti el. Ezeket a lekérdezéseket is transzparens módon iterációt shard térképre által hivatkozott összes shardlets-ban tárolt adatok. A rugalmas adatbázis API oldaltörések kereszt-shard le egy sorozat (egy, az egyes adatbázisok) egyes lekérdezések lekérdezése, és majd egyesíti az eredményeket. További információkért lásd: a lap [több shard lekérdezése] a Microsoft webhelyén.
  >
  >
* A ugyanazt a shard térképhez tartozó shardlets tárolt adatok ugyanazon séma kell rendelkeznie. Például nem mutat, néhány bérlői adatokat tartalmazó shardlets és egyéb shardlets termékinformációk tartalmazó lista shard térkép létrehozásához. Ez a szabály nem kényszeríti ki a rugalmas adatbázis, de az adatok kezelése, és rendkívül bonyolult, ha minden shardlet másik sémát lekérdezése válik. Az imént említett példában jó megoldás, ha a két lista shard maps: egy bérlő adatokat, majd egy másikat, mutató termékinformációk hivatkozik. Ne feledje, hogy ugyanazt a shard különböző shardlets tartozó adatok tárolhatók.

  > [!NOTE]
  > A rugalmas adatbázis API kereszt-shard lekérdezési funkcionalitás minden shardlet a shard leképezés ugyanazon séma tartalmazó függ.
  >
  >
* Tranzakciós műveletek csak adatokat, amelyek ugyanazt a shard belül és között szilánkok nem tartják támogatottak. Tranzakciók is kiterjedhet shardlets, mindaddig, amíg a ugyanazt a shard részét képezik. Ezért ha az üzleti logikát tranzakciók elvégzéséhez szüksége van, vagy az érintett adatok tárolása a ugyanazt a shard vagy valósítja meg a végleges konzisztencia. További információkért lásd: a [adatok konzisztencia ismertetése].
* Helyezze a szilánkok megközelíti a felhasználókat, akik e szilánkok adatait (azaz földrajzi-keresse meg a szilánkok). Ezt a stratégiát csökkenthető a késleltetés.
* Ne használjon magas keverékével aktív (csatlakozási pontokhoz) és viszonylag inaktív szilánkok. Próbálja meg a betöltési szilánkok egyenlően elosztva. Előfordulhat, hogy a shardlet kulcsok kivonatoláshoz.
* Ha szilánkok földrajzi megkeresése, győződjön meg arról, hogy a kivonatolt kulcsokat tárolja a felhasználói adatok közel szilánkok tárolt shardlets hozzárendelését.
* Jelenleg csak korlátozott készlete, típusok támogatottak shardlet kulcsként; SQL-adatok *int, bigint, varbinary,* és *uniqueidentifier*. Az SQL *int* és *bigint* típusok vannak rendelve a *int* és *hosszú* adattípusok C# nyelven íródtak, és ugyanazt a tartományt. Az SQL *varbinary* típus használatával kezelhető egy *bájt* C#, és az SQL tömb *uniqueidentier* típus megfelel-e a *Guid* az osztály a .NET-keretrendszer.

Foglalja, rugalmas adatbázis lehetővé teszi a rendszer hozzáadása és eltávolítása a szilánkok, az adatok mennyisége zsugorítja és -növelés. Az az Azure SQL Database Elastic Database ügyféloldali kódtár API-k lehetővé teszik, hogy az alkalmazások, és hozhat létre és szilánkok dinamikusan törlése (a shard térkép kezelőt frissíti, transzparens módon). A szilánkok eltávolítása azonban nem egy felülíró műveletet igénylő is, hogy a shard az összes adat törlése.

Ha egy alkalmazás egy shard felosztása két külön szilánkok vagy szilánkok kombinálhatja, rugalmas külön vegyes egyesítéses szolgáltatást nyújt. Ez a szolgáltatás a felhőben üzemeltetett szolgáltatás (amely léteznie kell a fejlesztő) fut, és biztonságosan szilánkok közötti áttelepítése. További információkért lásd a témakör [méretezés, a rugalmas adatbázis vegyes egyesítéses eszközzel] a Microsoft webhelyén.

## <a name="partitioning-strategies-for-azure-storage"></a>Az Azure Storage particionálási stratégia
Az Azure tárolási adatkezelésben négy absztrakt entitásokat biztosít:

* A Blob Storage a strukturálatlan objektumadatokat tárolja. Egy blob állhat bármilyen szövegből vagy bináris adatból, lehet például egy dokumentum, egy médiafájl vagy egy alkalmazástelepítő. A Blob Storage más néven Objektumtárnak is hívható.
* A Table Storage a strukturált adatkészleteket tárolja. A Table Storage a NoSQL-kulcsattribútumok adattára, amely gyors fejlesztési lehetőségeket és nagy adatmennyiségek gyors elérését biztosítja.
* A Queue Storage megbízható üzenetküldést biztosít a munkafolyamat-feldolgozáshoz és a felhőszolgáltatás összetevői közötti kommunikációhoz.
* A File Storage közös tárterületet biztosít a szabványos SMB-protokollt használó örökölt alkalmazások számára. Az Azure virtuális gépek és a felhőszolgáltatás csatlakoztatott megosztásokon keresztül adatokat oszthatnak meg az alkalmazások összetevői között, a helyszíni alkalmazások pedig a fájlszolgáltatás REST API-ján keresztül hozzáférhetnek a megosztott fájladatokhoz.

A TABLE storage és a blob Storage tárolóban lényegében kulcs-érték tárolók, illetve strukturált és strukturálatlan adatok tárolásához optimalizált. Tárolási sorok lazán összekapcsolt, méretezhető alkalmazások létrehozásához egy olyan mechanizmus biztosítása. A TABLE storage, a fájlok tárolására, a blob-tároló és a tárolási sorok Azure storage-fiók kontextusában jönnek létre. Storage-fiókok redundancia három formáját támogatja:

* **Helyileg redundáns tárolás**, amely egyetlen adatközponton belül adatok három másolatot tart fenn. Ezt az űrlapot a redundancia hardverhiba szemben, de nem egy olyan vészhelyzet esetén, amely magában foglalja a teljes adatközpont elleni védelmet nyújt.
* **Zónaredundáns tárolás**, ugyanabban a régióban belül különböző üzemeltetésében (vagy két földrajzilag Bezárás régióban) terjednek adatok három másolatot tart fenn, amelyek. Ezt az űrlapot a redundancia egyetlen adatközponton belül, de nem elleni vészhelyzetek ellen lehet védekezni nagy méretű hálózati bontja a kapcsolatot, amelyek hatással vannak az egész régió. Vegye figyelembe, hogy zónaredundáns tárolás jelenleg csak a blokkblobokhoz érhető el.
* **Georedundáns tárolás**, amely az adatok hat másolatot tart fenn: egy régió tartozik (a helyi régió) három példányban, és egy másik három példányban egy távoli régióban. Ezt az űrlapot a redundancia a legmagasabb szintű vész-helyreállítási védelmet nyújt.

Microsoft Azure Storage méretezhetőségi célok van közzétéve. További információkért lásd: a lap [Azure Storage méretezhetőségi és Teljesítménycélok] a Microsoft webhelyén. A teljes tárfiókok kapacitásával jelenleg legfeljebb 500 TB lehet. (Ez magában foglalja a tárolókészletben levő adatok tartott a table storage, a file storage blob-tároló, valamint a tároló várólista a tranzakció kimenetelére várva függőben lévő üzenetek).

A kérelem maximális (feltéve, hogy egy 1 KB-os entitás, blob vagy üzenet mérete) tárfiókok esetén ez 20 000 kérelmek / másodperc. A tárfiók legfeljebb 1000 iops-értéket (8 KB-nál) fájlmegosztás rendelkezik. Ha a rendszer várhatóan meghaladja ezt a korlátot, fontolja meg a betöltési particionálás több tárfiókok között. Egy Azure-előfizetéssel 200 storage-fiókokat hozhat létre. Vegye figyelembe azonban, hogy ezek a korlátozások idővel változhatnak.

## <a name="partitioning-azure-table-storage"></a>Az Azure table storage particionálás
Az Azure table storage egy kulcs-érték tárolóban, a particionálás ellátására van kialakítva. Egy partíció összes entitásának tárolja, és a partíciók belsőleg kezeli az Azure table storage. Minden entitás, amely a táblában tárolt meg kell adnia egy kétlépéses kulcsot, amely tartalmazza:

* **A partíciós kulcs**. Ez az egy karakterláncértéket, amely meghatározza, melyik partíció Azure table storage helyezi el az entitás. Az azonos partíciókulcsú valamennyi entitást tartalmazó partícióra tárolódnak.
* **A sorkulcs**. Ez az egy másik karakterlánc-érték, amely azonosítja az entitást a partíción belül. Egy partíció összes entitásának alapján rendezi a lexically, növekvő sorrendben, ezt a kulcsot. A partíciós kulcs/sor billentyűkombinációt minden egyes entitásnál egyedinek kell lennie, és a hossza legfeljebb 1 KB lehet.

Az adatokat, hogy egy entitás többi alkalmazás által meghatározott mezőből áll. Nem adott sémák lépnek érvénybe, és minden egyes sorára tartalmazhat egy másik alkalmazás által definiált mezők csoportját. A csak korlátozása, hogy egy entitás (beleértve a partíció- és sorfejlécek kulcsok) maximális méretének jelenleg 1 MB. A maximális tábla mérete 200 TB, bár ezeket az adatokat a későbbiekben változhat. (Ellenőrizze a lap [Azure Storage méretezhetőségi és Teljesítménycélok] kapcsolatos legfrissebb adatokat a Microsoft webhelyén.)

Ha tárolására, amelyek mérete meghaladja a kapacitását entitásokat próbál, fontolja meg több táblákba felosztásával őket. A mezők felosztani a csoportokat, amelyek a legnagyobb valószínűség együttesen érhető el a vertikális particionálást használnak.

A 7. ábrán egy példa storage-fiók (a Contoso adatok) logikai szerkezetének egy fiktív e-kereskedelmi alkalmazás jelennek meg. A tárfiók három táblát tartalmaz: ügyféladatok, termékinformációk és rendelés adatai. Minden tábla több partíciót tartalmaz.

Az ügyféladatok táblázatban az adatok particionálása megfelelően a város, amikor az ügyfél található, és a sorkulcs az ügyfél-azonosítót tartalmaz. A termékinformációk tábla termékkategóriák és a termékek particionáltak, és a sorkulcs a termék számát tartalmazza. A sorrend Info tábla rendelések particionáltak a dátum, amelyen helyezték, és a sorkulcs Megadja azt az időtartamot, a sorrendben érkezett. Vegye figyelembe, hogy minden adatot a sorkulcs minden partíció van rendezve.

![A táblák és a partíciók egy példa storage-fiók](./images/data-partitioning/TableStorage.png)

*7. ábra. A táblák és a partíciók egy példa storage-fiók*

> [!NOTE]
> Az Azure table storage időbélyegmezővel is hozzáadja minden entitáshoz. A Timestamp típusú mező a table storage tartja fenn, és minden alkalommal, amikor az entitást módosító és visszaírását a partíció frissül. A table storage szolgáltatás ezt a mezőt egyidejű hozzáférések optimista végrehajtásához használja. (Minden alkalommal, amikor egy alkalmazás ír egy entitás vissza a table storage a table storage szolgáltatás összehasonlítja az értéket az entitásban jelenleg írt Timestamp az értéket, amelyet a table storage használatban van. Ha az érték nem egyezik, az azt jelenti, hogy egy másik alkalmazás kell rendelkeznie az entitás óta módosított legutóbb beolvasott, és az írási művelet sikertelen lesz. Az Ön saját kódját a mező nem módosítható, és ez a mező értékét nem adja meg, amikor létrehoz egy új entitást.
>
>

Az Azure table storage a partíciós kulcs használ az adatok tárolásához, hogyan lehet. Egy entitás egy táblát ad hozzá egy korábban nem használt partíciós kulccsal, ha az Azure table storage új partíciót hoz létre ehhez az entitáshoz. Egyéb az azonos partíciókulcsú entitások partícióra tárolódnak.

Ez az eljárás egy automatikus kibővített stratégia ténylegesen alkalmazza. Mindegyik partíció egy Azure-adatközpontban annak biztosítására, hogy a lekérdezéseiben, amelyeket egyetlen partíció adatainak lekérése gyorsan futtassa a egy kiszolgálón tárolja. A különböző partíciók azonban több kiszolgálón is terjeszthetők. Emellett egyetlen kiszolgáló üzemeltethet több partíciót, ha ezek a partíciók korlátozott mérete.

Az entitások az Azure table storage tervezésekor, vegye figyelembe a következő szempontokat:

* A partíciós kulcs és a sor kulcsértékei kiválasztott kell áll az adatok elérése a. Válassza ki a partíciós kulcs/sor billentyűkombinációt, amely támogatja a legtöbb, a lekérdezések. A leghatékonyabb lekérdezések le adatokat a partíciós kulcs és a sorkulcs megadásával. Adja meg a partíciós kulcs és a sor kulcsok számos lekérdezések végezheti el egy olyan partíciót vizsgálatát. Ez a viszonylag gyors, mert az adatok sorok kulcs sorrendjének használatban van. Lekérdezések mely partíció vizsgálata nem ad meg, ha a partíciós kulcs Azure table storage megvizsgálja az adatok minden partíció lehet szükség.

  > [!TIP]
  > Egy entitás egy természetes kulccsal rendelkezik, ha használható a partíciós kulcs, és adja meg a sorkulcs egy üres karakterlánc. Ha egy entitás két tulajdonságait tartalmazó összetett kulccsal rendelkezik, jelölje ki a slowest változó tulajdonságot a partíciós kulcs, míg a másik a sorkulcs. Ha egy entitás legfeljebb két fő tulajdonságokkal rendelkezik, a Tulajdonságok összefűzése segítségével a partíció- és sorfejlécek kulcsait biztosítja.
  >
  >
* Ha rendszeresen végre lekérdezések, amelyek adatokat kereshet a partíció- és sorfejlécek kulcsok eltérő mezőkkel, vegye fontolóra a [index táblázat mintát].
* Ha készítése a partíciókulcsok használatával a monoton növekvő vagy csökkenő sorrendben (például "0001", "0002", "0003", és így tovább) minden partíció csak korlátozott mennyiségű adat tartalmazza, majd az Azure table storage is fizikailag ezek a partíciók egy csoportba a a ugyanazon a kiszolgálón. Ez az eljárás feltételezi, hogy az alkalmazás valószínűleg partíciók (lekérdezések) összefüggő számos különböző lekérdezések végrehajtásához, és ebben az esetben van optimalizálva. Ezt a módszert azonban egy kiszolgálón arra irányul, mert az új entitások minden Beszúrások valószínűleg több olyan end vagy egyéb összefüggő tartományt koncentrált elérési pontokhoz való vezethet. Méretezhetőség is csökkentheti. Egyenletesen a terhelés több kiszolgáló között, fontolja meg a partíciós kulcs, hogy a feladatütemezési több véletlenszerű kivonatoláshoz.
* Azure table storage támogatja a tranzakciós műveletek ugyanahhoz a partícióhoz tartozó entitások is szerepelnek. Ez azt jelenti, hogy egy alkalmazás végezheti több insert, update, delete, csere vagy egyesítési műveletek atomi egységként (feltéve, hogy a tranzakció nem tartalmazza a 100-nál több entitásokat és a kérelem hasznos nem haladhatja meg a 4 MB). Több partíción átnyúló műveletek nem tranzakciós, és előfordulhat, hogy meg kell valósítania a végleges konzisztencia szerint a [adatok konzisztencia ismertetése]. A table storage és a tranzakciók kapcsolatos további információkért lépjen a lapra [entitás csoport tranzakciók végrehajtása] a Microsoft webhelyén.
* Adja meg a lépésköz legyen a partíciós kulcs alapos figyelmet a következő okok miatt:
  * Minden entitás ugyanazzal a partíciókulccsal használata azt eredményezi, a table storage szolgáltatás egy kiszolgálón tartott egy nagy partíció létrehozásához. Ez megakadályozza, hogy kiterjesztése, és a terhelés dokumentum egy kiszolgálón. Ennek köszönhetően ez a megközelítés alkalmas csak rendszerek által kezelt entitások kis számú. Azonban ez a megközelítés győződjön meg arról, hogy valamennyi entitást részt vehetnek-e entitás csoport tranzakciók.
  * Minden entitás egyedi partíciós kulcs használata azt eredményezi, a table storage szolgáltatás minden egyes entitásnál, valószínűleg eredményezve kis partíciók (attól függően, hogy az entitások méretét) számos különböző partíció létrehozásához. Ez a megközelítés több méretezhető, mint a egyetlen partíciókulcsok használatával, de entitás csoport tranzakciók nem lehetséges. Emellett beolvasása egynél több entitáskészlet lekérdezések előfordulhat, hogy tartalmaz, amely egynél több kiszolgálón történő olvasás. Azonban ha az alkalmazás hajt végre a lekérdezések, monoton feladatütemezési segítségével a partíciós kulcsok létrehozása segíthet optimalizálni a lekérdezéseket.
  * A partíciós kulcs egy részhalmazát entitások közötti megosztás lehetővé teszi, hogy a csoport kapcsolódó bejegyzései szerepelnek a partícióra. Entitás csoport tranzakciók segítségével végrehajtható műveletek, például a kapcsolódó entitásokból, és a kapcsolódó entitásokból készlete fetch lekérdezések egyetlen kiszolgáló elérésével lehet teljesíteni.

Adatok az Azure table storage-ban partícionálásra vonatkozó további információkért lásd: a cikk [az Azure storage táblázat kialakítási útmutató] a Microsoft webhelyén.

## <a name="partitioning-azure-blob-storage"></a>Az Azure blob storage particionálás
Az Azure blob-tároló lehetővé teszi az nagyméretű bináris objektumok – jelenleg legfeljebb 5 TB-nál a blokkblobokhoz vagy lapblobokra 1 TB. (A legfrissebb információkért nyissa meg a lap [Azure Storage méretezhetőségi és Teljesítménycélok] a Microsoft webhelyén.) A forgatókönyvek használhatók, mint ahol kell streaming blokkblobokat segítségével feltölteni, vagy töltse le a nagy adatmennyiségek gyors. Ahelyett, hogy a soros hozzáféréshez, az adatokat a különböző részeinek véletlenszerű igénylő alkalmazások lapblobokat használnak.

Minden egyes blob (blokk vagy oldal) egy tároló az Azure-tárfiók használatban van. Tárolók használatával kapcsolódó blobok, amelyek az ugyanazon biztonsági követelmények csoportban. Ez a csoportosítás akkor logikai, hanem fizikai. Egy tároló belül minden egyes blob egyedi névvel rendelkezik.

A partíció egy blob kulcsa fiók nevét, a tároló neve + a blob neve. Ez azt jelenti, hogy minden egyes blob saját partícióval rendelkezhet, ha a blob terhelése megköveteli azt. Blobok terjeszthető környezetekben, sok kiszolgálón ahhoz, hogy terjessze ki őket a hozzáférést, de egyetlen blob csak egyetlen kiszolgáló szolgálhatók ki. 

Egy egyetlen blokkot (blokkblob) vagy a lap (az oldalakra vonatkozó blob) írása műveleteit atomi, de műveletek, amelyek több blokkolja, a lapok vagy a BLOB nem. Ha blokkolja, a lapokat és a blobok keresztül az írási műveletek végrehajtása során, győződjön meg arról, hogy konzisztencia van szüksége, vegye ki egy írási zárolás a blob címbérlet használatával.

Az Azure blob storage tárolók átviteli sebességre legfeljebb 60 MB második vagy 500 kérelmek minden egyes blob másodpercenkénti száma. Ha ezek a korlátozások határértékek túllépését várhatóan, és a blobadatokat viszonylag statikus, fontolja meg bináris objektumok replikálása az Azure Content Delivery Network használatával. További információkért lásd: a lap [Azure Content Delivery Network] a Microsoft webhelyén. További útmutatás és szempontokat, [használata Azure Content Delivery Network].

## <a name="partitioning-azure-storage-queues"></a>Az Azure storage várólisták particionálás
Az Azure storage-üzenetsorok lehetővé teszik-folyamatai közötti aszinkron üzenetkezelési végrehajtásához. Azure-tárfiók várólisták tetszőleges számú tartalmazhat, és egyes várólisták korlátlan számú üzenetet tartalmazhat. Egyetlen korlátozás a hely, amely a tárfiókban lévő érhető el. Egy egyszeri üzenetek maximális mérete 64 KB. Ha nagyobb, mint ez üzenetek van szüksége, fontolja meg helyette az Azure Service Bus-üzenetsorok.

Minden egyes tárolási várólista, amely tartalmazza azt a tárfiókon belül egyedi névvel rendelkezik. Azure particionálja a várólisták neve alapján. Ugyanazon a várólistán üzenetek egy adott kiszolgáló vezérli partícióra vannak tárolva. Különböző várólisták kezelhető a terhelés elosztása érdekében különböző kiszolgálókon. Lefoglalása a várólisták kiválasztásával kiszolgálók az alkalmazások és a felhasználók számára transzparens.

 Egy nagy méretű alkalmazásban ne használjon ugyanazon a várólistán tárolási az alkalmazás összes példányát, mert ez a megközelítés előfordulhat, hogy a kiszolgáló, amelyen az interaktív terület lesz a várólista. Ehelyett használjon különböző sorok különböző funkcionális területek az alkalmazás. Az Azure storage várólisták támogatja a tranzakciókat, így arra utasíthatja a különböző üzenetsorok üzenetek kell jelentősek üzenetküldéssel konzisztencia.

Egy Azure storage üzenetsorába kezelni tud a legfeljebb 2000 üzenetek száma másodpercenként.  Ennél nagyobb arányban üzenetek feldolgozásához, javasolt több várólisták létrehozására. Például egy globális alkalmazásban hozzon létre külön tárüzenetsort külön tárfiókokban minden régióban futtató alkalmazáspéldányok kezelésére.

## <a name="partitioning-strategies-for-azure-service-bus"></a>Azure Service Bus particionálási stratégia
Azure Service Bus egy üzenet broker használatával a Service Bus-üzenetsor vagy témakör küldött üzenetek kezeléséhez. Alapértelmezés szerint üzenetsor vagy témakör küldött összes üzenet kezeli a ugyanazon üzenet broker folyamat. Ez az architektúra korlátozása állíthat be a teljes átviteli képessége – az üzenet-várólista. Azonban Ön is is partícióazonosító üzenetsor vagy témakör létrehozásakor. Úgy, hogy ehhez a *EnablePartitioning* az üzenetsor vagy témakör leírást tulajdonságának *igaz*.

A particionált üzenetsor vagy témakör több töredék biztonsági, amelyek mindegyike egy külön üzenettároló, valamint az üzenet broker van felosztva. A Service Bus létrehozását és kezelését, ez a töredék felelősséget. Egy alkalmazás particionált üzenetsor vagy témakör üzenetet küldi, a Service Bus egy kódrészletet az adott üzenetsor vagy témakör az üzenet rendel. Amikor egy alkalmazás egy üzenetsorból vagy előfizetés üzenetet kap, a Service Bus ellenőrzi a következő elérhető üzenetet minden egyes töredék, és átadja az alkalmazás feldolgozásra.

Ez a struktúra segít a terhelés szétosztását üzenet brókerek és üzenettároló, hatékonyabb méretezhetőséget és rendelkezésre állás javítása. Ha egy üzenet broker vagy az üzenet tároló átmenetileg nem érhető el, a Service Bus lehet üzeneteket beolvasni a fennmaradó rendelkezésre álló töredék egyikéhez.

A Service Bus rendel hozzá egy üzenet egy kódrészletet az alábbiak szerint:

* Ha az üzenet egy munkamenet tartozik, az összes üzenetek ugyanazt az értéket a * ugyanazon töredék küldött munkamenet-azonosító * tulajdonság.
* Ha az üzenet nem tartozik egy munkamenetet, de a küldő rendelkezik a megadott értéket a *PartitionKey* tulajdonság, majd ugyanazzal a összes üzenet *PartitionKey* értéket küldött ugyanazon töredék.

  > [!NOTE]
  > Ha a *SessionId* és *PartitionKey* mindkét megadva a tulajdonságai, akkor ugyanazt az értéket be kell állítani, vagy az üzenet vissza lesznek utasítva.
  >
  >
* Ha a *SessionId* és *PartitionKey* üzenet tulajdonságai nincsenek megadva, de ismétlődő érzékelés engedélyezve van, a *MessageId* tulajdonság használható. Összes üzenet azonos *MessageId* ugyanazon töredék a rendszer kéri.
* Ha az üzenetek nem tartalmaznak egy *munkamenet-azonosító, PartitionKey,* vagy *MessageId* tulajdonság, akkor a Service Bus rendel üzenetek töredék egymás után. Töredékkel nem érhető el, ha a Service Bus helyezi át a Tovább gombra. Ez azt jelenti, hogy az üzenetkezelési infrastruktúrára egy ideiglenes hiba nem okoz az üzenet-küldési művelet sikertelen lesz.

Vegye figyelembe az alábbiakat, ha meghatározásakor vagy a Service Bus-üzenetsor vagy témakör particionálásáról:

* Service Bus-üzenetsorok és témakörök jönnek létre a Service Bus-névtér hatókörén belül. A Service Bus jelenleg lehetővé teszi, hogy legfeljebb 100 particionált várólisták vagy olyan témakörök / névtér.
* Minden egyes Service Bus-névtér írja elő a rendelkezésre álló erőforrások, például előfizetésszám száma párhuzamos küldés számát, a témakör az kvótái, és részesüljön kérelmek / másodperc, és a létrehozható egyidejű kapcsolatok maximális száma. Ezek mely százalékértékénél kéri a Microsoft webhelyén, a lapon szerepelnek [Service Bus kvóták]. Ha várhatóan a következő korlátozásokkal ezeket az értékeket, a saját üzenetsorok és témakörök további névteret létrehozása, és a munka elosztva a névterek. Globális alkalmazásokban, például különálló névterek létrehozása minden régióban, és a alkalmazáspéldányok az üzenetsorok és témakörök a legközelebbi névtér használatára konfigurálja.
* Egy tranzakció részeként küldött üzenetek meg kell adnia egy partíciókulcsot. Ez lehet egy *SessionId*, *PartitionKey*, vagy *MessageId* tulajdonság. Ugyanabban a tranzakcióban részeként küldött összes üzenet kell megadni ugyanazzal a partíciókulccsal, mert azok a ugyanazon üzenet broker folyamat kell kezelnie. Üzenetek küldése nem különböző várólisták vagy olyan témakörök ugyanazon a tranzakción belül.
* A particionált üzenetsorok és témakörök automatikusan törlésekor üresjárati válnának nem konfigurálható.
* A particionált üzenetsorok és témakörök jelenleg használható a speciális Message Queuing protokoll (AMQP) platformfüggetlen vagy hibrid megoldások készítésekor.

## <a name="partitioning-strategies-for-documentdb-api"></a>A DocumentDB API particionálási stratégia
Azure Cosmos DB egy NoSQL-adatbázis, amely dokumentumokat tárolhat a [DocumentDB API][documentdb-api]. A dokumentum egy Cosmos DB adatbázisban egy objektum vagy egyéb adatok JSON-szerializált ábrázolását. Nincsenek rögzített sémák kerül minden alkalom, azzal a különbséggel, hogy minden dokumentumnak tartalmaznia kell egy egyedi azonosítót.

Dokumentumok gyűjteményekbe vannak rendezve. Kapcsolódó dokumentumok együtt egy gyűjteményen belül csoportosíthatja. Például az állapotinformációkat bejegyzéseket, tárolhatja minden blogbejegyzés tartalmát a gyűjtemény-dokumentumként. Az egyes tulajdonos gyűjteményeket is létrehozhat. Azt is megteheti több-bérlős alkalmazásokban, például a rendszer, ahol a különböző szerzők szabályozza, és kezelheti a saját blogbejegyzések is Szerző blogok partícióazonosító és minden egyes Szerző külön gyűjtemények létrehozása. A gyűjtemények lefoglalt tárhely rugalmas és zsugorításával vagy igény szerinti.

A dokumentum gyűjtemények lehetővé természetes egyetlen adatbázisban lévő adatok particionálása. Belső egy Cosmos DB adatbázisban több kiszolgáló is kiterjedhet, és megkísérelhetik a terhelés terjednek gyűjtemények elosztásával kiszolgáló között. A legegyszerűbben horizontális megvalósítása után minden shard a gyűjtemény létrehozásához.

> [!NOTE]
> Minden egyes Cosmos DB-adatbázis egy *teljesítményszintet* , amely meghatározza, hogy lekéri a mérete. A teljesítményszintet társítva van egy *kérelem egység* sávszélesség-korlátjának (RU). A sávszélesség-korlátjának RU meg fenntartott és a rendelkezésre álló kizárólagos adott gyűjtemény által használt erőforrások mennyisége határozza meg. A gyűjtemény költségét attól függ, hogy az adott gyűjtemény kiválasztott teljesítményszint szükséges. Minél nagyobb teljesítményt szint (és a sávszélesség-korlátjának RU) minél nagyobb a kell fizetni. Az Azure portál használatával módosíthatja a gyűjtemény teljesítményszintjét. További információkért lásd a [teljesítményszintek az Adatbázisba az Cosmos] cikket a Microsoft webhelyén.
>
>

Összes adatbázis egy Cosmos-adatbázis adatbázis-fiók környezetében jönnek létre. Egy olyan fiók több adatbázisok tartalmazhat, és meghatározza azokat a területeket az adatbázisok jön létre. Minden felhasználói fiókhoz is érvényesíti a saját hozzáférés-vezérlést. Használhatja a földrajzi Cosmos DB fiókokat-keresse meg a szilánkok (gyűjtemények adatbázisok belül) megközelíti a felhasználók, akik azok eléréséhez, és a korlátozások érvényesítése, hogy csak azokat a felhasználók kapcsolódhatnak őket.

Minden Cosmos DB rendelkezik, amely korlátozza a számú adatbázisból és gyűjteményeket, amelyek tartalmazhat, és elérhető a dokumentum tárolókapacitást. További információkért lásd: [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések][azure-limits]. Elméletileg lehetséges, hogy alkalmazza a rendszer, ahol minden szilánkok tartozik ugyanarra az adatbázisra, ha előfordulhat, hogy elérte a fiók a tárolási kapacitásán belül.

Ebben az esetben szükség lehet további fiókok és adatbázisok létrehozása, és a szilánkok szét ezeket az adatbázisokat. Azonban akkor is, ha a tárolási kapacitás-adatbázis eléréséhez valószínűleg ajánlott egy több adatbázis használatára. Ennek oka az egyes adatbázisok saját rendelkezik a felhasználókat és engedélyeket, és a mechanizmus segítségével elkülönítése adatbázis-alapú gyűjtemények elérésére.

8. ábra a DocumentDB API általános szerkezetét mutatja be.

![A DocumentDB API szerkezete](./images/data-partitioning/DocumentDBStructure.png)

*8. ábra.  A DocumentDB API architektúra szerkezete*

A feladat a megfelelő shard kérelmek általában közvetlen alkalmazásával segítse a saját leképezés mechanizmus az adatokat, amelyek meghatározzák a shard kulcs bizonyos attribútumai alapján az ügyfélalkalmazás. 9. ábra mutatja a két DocumentDB API-adatbázisok esetén két szilánkok működő gyűjteményeket tartalmazó. Az adatok szilánkos egy bérlő-azonosító szerint, és tartalmazza az adatokat egy adott bérlő számára. Az adatbázisok külön Cosmos DB fiókok jönnek létre. Ezeket a fiókokat és a bérlők számára, amelyek adatokat tartalmaznak ugyanabban a régióban találhatók. Az ügyfélalkalmazás az útválasztási logika a bérlő azonosítója a shard kulcsként használja.

![A DocumentDB API-jával végrehajtási horizontális](./images/data-partitioning/DocumentDBPartitions.png)

*9. ábra. A DocumentDB API-jával végrehajtási horizontális*

A DocumentDB API adatok particionálása módjának, vegye figyelembe a következő szempontokat:

* **A DocumentDB API-adatbázis számára elérhető erőforrások használata tekintetében a fiók sablonkérelem**. Az egyes adatbázisok tárolására képes a gyűjtemények számos (ebben az esetben korlátozás van), és egyes gyűjtemények társítva, amely szabályozza a RU sávszélesség-korlátjának (fenntartott átviteli sebességet), hogy a gyűjtemény egy teljesítményszint szükséges. További információ: [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések].
* **Ügyeljen rá, hogy a dokumentum a gyűjteményt, amelyben tárolt belül egyedi azonosítására használható egy attribútummal kell rendelkeznie,**. Ez az attribútum található a szilánkcímtárban kulcs, amely meghatározza, mely a gyűjtemény kérelemfejléceket tárol, a dokumentum eltér. Egy gyűjtemény tartalmazhat nagyszámú dokumentumok. Elméletileg azt csak korlátozza a maximális időtartamot, a dokumentum azonosítóját. A dokumentum azonosító legfeljebb 255 karakterből állhat.
* **Az összes dokumentum elleni műveleteket egy tranzakció keretén belül. Tranzakciók hatóköre a gyűjteményt, amelyben a dokumentum tartalmazza.** Egy művelet meghiúsul, ha a munka még hajtott végre vissza lesz állítva. Míg egy dokumentumot egy művelet függvényében, minden egyes helyadatbázisokban végrehajtott módosításokat a pillanatkép szintű elkülönítés vonatkoznak. Ez a módszer biztosítja azt, hogy ha például egy új dokumentum létrehozása sikertelen, akik egy időben kérdezi le az adatbázis egy másik felhasználó kérést nem jelenik meg egy részleges dokumentumot, majd eltávolítani.
* **A gyűjtemény szintjén is hatóköre az adatbázis-lekérdezések**. Egyetlen lekérdezés adatainak lekérése is csak egy gyűjtemény. Ha adatainak lekérése több gyűjteményre van szüksége, kell egyes gyűjtemények külön-külön és -lekérdezés egyesítése az eredményeket az alkalmazás kódjában.
* **A DocumentDB API adatbázisok által támogatott programozható elemek, amelyek az összes találhatók dokumentumok mellett egy gyűjtemény**. Ezek közé tartozik a tárolt eljárások, felhasználó által definiált függvények és eseményindítók (JavaScript nyelven írt). Ezek az elemek összes dokumentumának belül ugyanaz a gyűjtemény. Ezenkívül ezek az elemek futtatása (esetén egy eseményindítót, amely akkor következik be, mert eredménye egy létrehozása, törlése, vagy műveletet hajt végre a dokumentum cseréje), a környezeti tranzakció hatókörén belül, vagy (esetén tárolt eljárás új tranzakció elindítása futtatott az explicit ügyfélkérés miatt). A kód egy programozható elemben kivételt jelez, ha a tranzakció vissza lesz állítva. Tárolt eljárások és eseményindítók használhatja az integritásra és a dokumentumok között konzisztencia fenntartása, de ezeket a dokumentumokat ugyanaz a gyűjtemény részének kell lennie.
* **A gyűjtemények, melyet a adatbázisokban tárolásához valószínűleg nem haladja meg a teljesítményszintet gyűjtemények által megadott átviteli sebességének korlátai kell**. Lásd: Azure Cosmos DB egység kérjen további informationm][cosmos-db-ru]. Ha várhatóan a működés felső korlátjának elérése, fontolja meg a gyűjtemények gyűjteményenként a terhelés csökkentése érdekében különböző fiókok az adatbázisok közötti felosztásával.

## <a name="partitioning-strategies-for-azure-search"></a>Az Azure Search particionálási stratégia
Adatok keresése a gyakran és sok webes alkalmazások által biztosított feltárása az elsődleges módszer. Ennek segítségével a felhasználók, erőforrások gyorsan (például az e-kereskedelmi alkalmazás-termékek) kombinációit keresési feltételek alapján található. Az Azure Search szolgáltatás teljes szöveges keresési lehetőségeket biztosítanak a webes tartalom, és a szolgáltatások, mint a találatok és a jellemzőalapú navigáció közelében alapuló begépelt, javasolt lekérdezések tartalmaz. Ezek a képességek teljes leírását az oldalon érhető el [Azure Search újdonságai?] a Microsoft webhelyén.

Az Azure Search kereshető tartalom adatbázis a JSON-dokumentumokként tárolja. Megadhatja, hogy a kereshető mezők meg ezeket a dokumentumokat és e definíciókat meg az Azure Search index. Ha a keresési kérelem elküldése, Azure Search használja a megfelelő indexek egyező elemek.

A versengés csökkentése érdekében az Azure Search által használt tárolási 1, 2, 3, 4, 6 vagy 12 partíciók oszthatók, és mindegyik partíció legfeljebb 6 alkalommal replikálható. A replikák számának szorzata partíciók számának a termék neve a *keresési egység* (SU). Azure Search egyetlen példányát maximum 36 SUS-t (12 partíciókkal rendelkező adatbázis csak legfeljebb 3 replikák) tartalmazhat.

A szolgáltatás az összes lefoglalt SU számlázása. Növekedésével a kötet kereshető tartalom nő vagy keresési kérelmek számát, SUS-t is hozzáadhat az Azure Search a megnövekedett terhelés kezelésére egy meglévő példányát. Az Azure Search magát a dokumentumok egyenletesen elosztja a partíciók. Nincs manuális particionálási stratégia jelenleg támogatott.

Mindegyik partíció tartalmazhatnak legfeljebb 15 millió dokumentumnál vagy 300 GB tárterületet (amelyik érték kisebb) foglalnak. Legfeljebb 50 indexek hozhat létre. A szolgáltatás teljesítményének változik, és a dokumentumok, a rendelkezésre álló indexek és a hálózati késés hatásait bonyolultságától függ. Átlagosan egy replikához (1 SU) kell tudni kezelni 15 lekérdezések / másodperc (QPS), bár javasolt végrehajtását átviteli pontosabb mérték beszerzése a saját adataival teljesítménymérésre. További információkért lásd: a lap [szolgáltatási korlátait, az Azure Search] a Microsoft webhelyén.

> [!NOTE]
> Adattípusok korlátozott számú kereshető dokumentumok, karakterláncok, a logikai, numerikus, dátum és idő data, és egyes földrajzi adatokat tárolhatja. További részletekért lásd: a lap [a támogatott adattípusokat (Azure Search)] a Microsoft webhelyén.
>
>

Korlátozott számú szabályozhatják, hogyan Azure Search particionálja a szolgáltatás minden példányának adatait. Azonban a globális környezetben valószínűleg jobb teljesítmény és csökkenteni a késleltetés és a további versengés particionálás a szolgáltatás a következő stratégiák egyikét követve:

* Hozzon létre egy Azure Search példányát minden egyes földrajzi régióban, és győződjön meg arról, hogy az ügyfélalkalmazások irányul a legközelebbi elérhető példányhoz. Ezt a stratégiát szükséges kereshető tartalom frissítéseit időben replikálódnak a szolgáltatás minden példányának.
* Hozzon létre két rétegből álló Azure Search:

  * Minden egyes régió, amely tartalmazza az adatokat, amelyek a leggyakrabban az adott régióban felhasználók férhetnek hozzá a helyi szolgáltatás. Felhasználók is közvetlen kérelmeket Itt a gyors, de korlátozott eredmények elérése érdekében.
  * A globális szolgáltatás, amely magában foglalja az összes adatot. Felhasználók utasíthatja a lassabb, de több teljes eredmény itt kérelmek.

Ezt a módszert akkor legmegfelelőbb, ha az adatokat, amelyben keres regionális jelentős eltérés van.

## <a name="partitioning-strategies-for-azure-redis-cache"></a>Az Azure Redis Cache particionálási stratégia
Azure Redis Cache a felhőben, amelyek a Redis-kulcs-érték tároló alapján egy megosztott gyorsítótárazási szolgáltatást biztosít. Mivel a név azt jelenti, Azure Redis Cache szándék szerint gyorsítótárazási megoldását. Használja azt, csak az átmeneti adatokat, és nem állandó adattárat. Alkalmazások, amelyek ténylegesen használják az Azure Redis Cache tudják is működjenek, ha a gyorsítótár nem érhető el. Azure Redis Cache elsődleges és másodlagos támogatja magas rendelkezésre állás biztosításához, de jelenleg korlátozza az 53 GB a gyorsítótár maximális méretét. Ha ennél több helyre van szüksége, létre kell hoznia további gyorsítótárak. További információkért lépjen a lapra [Azure Redis Cache] a Microsoft webhelyén.

A Redis-tárolóban particionálás magában foglalja az adatok felosztása a Redis szolgáltatás példánya között. Minden példány egyetlen partícióra számít. Azure Redis Cache kivonatolja a Redis-szolgáltatások egy homlokzati mögött, és nem teszi közzé őket közvetlenül. A legegyszerűbben úgy particionálás végrehajtására több Azure Redis Cache példány létrehozásához, és az adatok elosztva.

Egy azonosító (a partíciós kulcs), amely meghatározza, hogy melyik gyorsítótár tárolja az elem minden adatelemet társíthatja. Az ügyfél úgy az alkalmazáslogikát ezután használhatja ezt az azonosítót kérelmek átirányítása a megfelelő partíció. Ez a séma nagyon egyszerű, de a particionálási sémát változik (például, ha több Azure Redis Cache példányt jönnek létre), ha ügyfélalkalmazások módosítania kell újra kell konfigurálni.

Natív Redis (Azure Redis Cache nem) támogatja a kiszolgálóoldali particionálás Redis fürtszolgáltatás alapján. Ez a megközelítés a oszthatja az adatokat egyenletesen kiszolgáló között kivonatoló mechanizmus használatával. Minden egyes Redis-kiszolgáló tárolja a tartomány, amely tárolja a partíció, kivonatoló kulcsok leíró metaadatok és mely kivonatoló kapcsolatos kulcsok más kiszolgálókon a partíciók található információkat is tartalmaz.

Ügyfél-alkalmazások egyszerűen kérelmet küldeni a programban részt vevő Redis-kiszolgáló (valószínűleg a legközelebbi egy). A Redis-kiszolgáló megvizsgálja az ügyfélkérés. Ha helyileg feloldható, a kért műveletet hajt végre. Ellenkező esetben továbbítja, a kérelem továbbítása az a megfelelő kiszolgálót.

Ez a modell Redis Fürtszolgáltatás segítségével történik, és további részletes leírását lásd a a [Redis-fürt oktatóanyag] lap a Redis-webhelyen. A redis-fürtszolgáltatás az ügyfélalkalmazások számára transzparens. További Redis-kiszolgálók adhatók hozzá a fürtöt (és az adatok újra lehet particionálni) anélkül, hogy be újra az ügyfeleket.

> [!IMPORTANT]
> Azure Redis Cache jelenleg nem támogatja a Redis-fürtszolgáltatás. Ha szeretné végrehajtani ezt a módszert, az Azure-ral, majd meg kell valósítani a saját Redis kiszolgálók Redis telepítése Azure virtuális gépek egy csoportján és konfigurálásuk manuálisan. A lap [a CentOS Linux virtuális gép az Azure-ban futó Redis] a Microsoft webhelyén végigvezeti egy példa, amely azt ismerteti, hogyan hozza létre és konfigurálja az Azure virtuális gépként futó Redis csomópont.
>
>

A lap [particionálására: hogyan adatok több Redis-példány között] a Redis webhelyet biztosít a redis gyorsítótárral particionálás alkalmazásával kapcsolatos további információkat. Ez a szakasz többi feltételezi, hogy az ügyféloldali vagy a proxy támogatású particionálás webkiszolgálókból.

Azure Redis Cache adatok particionálása módjának, vegye figyelembe a következő szempontokat:

* Azure Redis Cache nem célja, hogy működjön, és a végleges tárolóban, ezért bármilyen particionálási sémát alkalmazza, az alkalmazás kódjában képesnek kell lennie lekérdezni az adatok egy helyre a rendszer a gyorsítótárban.
* Együtt a gyakran használt adatok ugyanazon partíció kell tartani. A redis egy hatékony kulcs-érték tároló, amely több magas szinten optimalizált mechanizmusok adatok rendszerezésére szolgál. Ezek a mechanizmusok az alábbiak egyike lehet:

  * Egyszerű karakterláncok (bináris adatok hossza legfeljebb 512 MB)
  * Összesített típuson például listák (amely működhet várólisták és a verem)
  * Beállítja a (rendezett és rendezetlen)
  * A kivonatok (amely is kapcsolódó mezők egy csoportba, például a elemek, amelyek megfelelnek a mezők objektum)
* Az összesített típusok lehetővé teszik a ugyanazzal a kulccsal sok kapcsolódó értéket hozzárendelni. Egy Redis-key listáját, be van állítva, vagy kivonatoló helyett a benne tárolt adatelemek azonosítja. Ezek a típusok Azure Redis Cache segítségével elérhetők, és ismerteti a által a [adattípusok] lap a Redis-webhelyen. Például részben az elektronikus kereskedelmi rendszer, amely nyomon követi az ügyfelek általi rendeléseket, mindegyik ügyfél részletes adatait tárolhatja a Redis-kivonat, amely a felhasználói azonosítóját. a kulccsal van Minden Kivonatoló azonosítók rendelés gyűjteménye tárolható az ügyfél. Egy külön Redis-készlet tárolására képes újra, a kivonatok strukturált, és az azonosító. a kulccsal definiált rendelések 10. ábrán látható, ez a struktúra. Vegye figyelembe, hogy Redis nem valósítja meg a hivatkozási integritás bármilyen, a fejlesztő felelőssége, hogy az ügyfelek és a rendeléseket kapcsolatának fenntartásához.

![A Redis-tároló megrendelések és azok adatai rögzítése javasolt struktúra](./images/data-partitioning/RedisCustomersandOrders.png)

*10. ábra. A Redis-tároló megrendelések és azok adatai rögzítése javasolt struktúra*

> [!NOTE]
> A Redis minden kulcs bináris adatok értékek (például a Redis-karakterláncok), és legfeljebb 512 MB adatokat is tartalmazhat. Elméletileg kulcs szinte bármilyen információkat is tartalmazhat. Azt javasoljuk azonban bevezetése egységes elnevezési kulcsok leíró, milyen típusú adatok, amelyek, amely azonosítja az entitást, de nem túl hosszú. Általános gyakorlatként javasolt, hogy az űrlap "entity_type:ID" kulcsok használja. Például az "felhasználói: 99" segítségével jelzik az azonosító 99 az ügyfél a kulcsot.
>
>

* A vertikális particionálás kapcsolódó információk tárolása különböző összesítések egy adatbázisban is létrehozható. Például egy e-kereskedelmi alkalmazás tárolhatja termékek egy Redis-kivonatot, és kevesebb mint egy másik gyakran használt részletes információk általánosan használt információ.
  Mindkét kivonatokat a azonos termékazonosító használható a kulcs részét. Használhat például "termék:  *nn* " (ahol  *nn*  termék azonosítója) az információkat és "product_details:  *nn*  "a részletes adatokat. Ezt a stratégiát, amely a legtöbb lekérdezésnél valószínűleg beolvasása adatok mennyisége csökkenthető.
* Egy Redis adatok tárolására, de vegye figyelembe, hogy a rendszer egy összetett és időigényes feladat particionálhatja. A redis-fürtszolgáltatás automatikusan adatok particionálhatja, de ez a funkció nem érhető el az Azure Redis Cache. Ezért amikor a particionálási sémát, próbálja hagyjon elegendő szabad hely a partíciókat úgy, hogy lehetővé teszik a várt adatnál növekedési adott idő alatt. Ne feledje azonban, hogy Azure Redis Cache célja, hogy adatok gyorsítótárazása ideiglenesen, illetve, hogy a gyorsítótárban tárolt adatok, hogy egy korlátozott élettartamáról, megadott idő-live (TTL) értéket. Viszonylag "volatile" adatok lehet, hogy az élettartam rövid, de a statikus adatok az élettartam sokkal hosszabb lehet. Kerülje a nagy mennyiségű hosszú élettartamú adatok tárolása a gyorsítótár, ha ezek az adatok mennyisége várhatóan töltse ki a gyorsítótárból. Megadhat egy kiürítés szabályzattal, amely eltávolítja az adatokat, ha a hely jelenleg egy prémium szintű Azure Redis Cache okoz.

  > [!NOTE]
  > Azure Redis Cache-gyorsítótár használata esetén az a megfelelő árképzési szint kiválasztásával adja meg (az 53 GB 250 MB) a gyorsítótár maximális méretét. Azonban az Azure Redis Cache-gyorsítótár létrehozása után nem növelése (vagy csökkentése) mérete.
  >
  >
* Redis kötegek és a tranzakciók nem terjedhetnek ki több kapcsolatot, a köteg vagy a tranzakció által érintett összes adatot kell maradnia ugyanabban az adatbázisban (szilánkok) így.

  > [!NOTE]
  > A Redis-tranzakció műveletei sorozatát nincs feltétlenül atomi. A tranzakció alkotó parancsok ellenőrzése és a sorba állított még futtatása előtt. Ha a hiba akkor fordul elő, ebben a fázisban, a rendszer törli a teljes sor. Azonban után a tranzakció küldése sikeres volt, a várólistán lévő parancsokat futtassa sorrendben. Ha a parancs sikertelen, a parancsnak csak leáll. A várólistában lévő összes előző és a következő parancsok megy végbe. További információkért látogasson el a [tranzakciók] lap a Redis-webhelyen.
  >
  >
* A redis támogatja egy atomi műveletek száma korlátozott. Csak ilyen típusú, amely támogatja a több kulcsok és értékek olyan MGET és MSET műveleteket. Kell visszaadnia MGET műveletek a megadott listán szereplő kulcsokra értékeit, és MSET műveletek kulcsok megadott listája értékek gyűjteménye tárolja. Ha szeretné használni ezeket a műveleteket, a MSET és MGET parancsok által hivatkozott kulcs-érték párok belül az azonos adatbázist kell tárolni.

## <a name="partitioning-strategies-for-azure-service-fabric"></a>Az Azure Service Fabric particionálási stratégia
Az Azure Service Fabric egy mikroszolgáltatások platform, amely egy futásidejű biztosít elosztott alkalmazások számára a felhőben. A Service Fabric támogatja a .net Vendég végrehajtható fájlok, az állapot nélküli és állapotalapú alkalmazások és szolgáltatások és tárolók. Állapotalapú szolgáltatások biztosítják a [megbízható gyűjtemény] [ service-fabric-reliable-collections] tartósan adatok egy kulcs-érték gyűjtemény belül a Service Fabric-fürt tárolására. Egy megbízható gyűjtemény partioning kulcsok kapcsolatos olyan stratégiák kapcsolatos további információkért lásd: [irányelvek és javaslatok az Azure Service Fabric megbízható gyűjtemények].

### <a name="more-information"></a>További információ
* [Azure Service Fabric áttekintése] bemutatja az Azure Service Fabric.
* [A Service Fabric megbízható szolgáltatások partícióazonosító] további információt az Azure Service Fabric megbízható szolgáltatásokat nyújt.

## <a name="partitioning-strategies-for-azure-event-hubs"></a>Az Azure Event Hubs particionálási stratégia

[Az Azure Event Hubs] [ event-hubs] olyan fürtrendszergazdáknak készült, nagy léptékű streaming és particionálás horizontális skálázás engedélyezése a szolgáltatás be van építve. Mindegyik felhasználó csak egy adott partícióra az üzenet-adatfolyam olvassa be. 

Az esemény-közzétevő csak a partíciókulcsot ismeri, azt a partíciót nem, amelyre az esemény közzé lesz téve. A kulcs és a partíció szétválasztása révén a küldőnek nem szükséges behatóan ismernie az alárendelt feldolgozási folyamatokat. (Egyben események lehetséges küldése közvetlenül egy adott partíció, de általában nem ajánlott.)  

Hosszú távú skálázási megfontolandó választja, a partíciók száma. Az eseményközpontok létrehozását követően nem módosíthatja a partíciók számát. 

Az Event Hubs partíciók használatával kapcsolatos további információkért lásd: [Mi az az Event Hubs?].

Rendelkezésre állás és a konzisztencia közötti kompromisszumot kapcsolatos problémák, lásd: [rendelkezésre állását és az Event Hubs következetes].

## <a name="rebalancing-partitions"></a>Partíciók újraelosztás
Miután a rendszer kiforrottá válik, és a használati minták jobban megismerte, lehetséges, hogy úgy, hogy a particionálási sémát. Például az egyes partíciók előfordulhat, hogy start, valamint a forgalom le aránytalanul nagy mennyiségű és forró, ami túl sok versengés válnak. Emellett, előfordulhat, hogy rendelkezik kellőképpen adatokat az egyes partíciók okozza, hogy megközelíti a tárolási kapacitás, ezek a partíciók a határain. A valamilyen okból, fontos néha egyensúlyba partíciók több egyenletesen a terhelést.

Bizonyos esetekben a adattároló rendszerek, amelyek nem nyilvánosan láthatóvá adatok kiszolgálókon elosztását vezérli automatikusan egyensúlyba partíciók rendelkezésre álló erőforrások keretein belül. Más helyzetekben újraelosztás felügyeleti feladatot, amely két szakaszból áll:

1. Annak megállapítása, az új particionálási stratégia meghatározása:
   * Előfordulhat, hogy partíciók kell vágási (vagy esetlegesen).
   * Ezek a partíciók adatok új partíciós kulcsok megtervezésével lefoglalni módját.
2. Az érintett adatok áttelepítése a régi particionálási sémát az újonnan létrehozott partíciók.

> [!NOTE]
> Kiszolgálók adatbázis gyűjtemények leképezése nem átlátszó, de továbbra is érhető el a tárolási kapacitás és átviteli sebességének korlátai Cosmos DB fiók. Ha ez történik, előfordulhat, hogy kell terveznie a particionálási sémát, és az adatok áttelepítéséhez.
>
>

Attól függően, hogy az adatok tárolási technológia, és az adatok tárolási rendszer kialakítása valószínűleg tudni át adatokat, amíg azok használja (online áttelepítés) partíciók között. Ha ez nem lehetséges, szükség lehet annak az érintett partíciók átmenetileg nem érhető el, amíg az adatok áthelyezett (kapcsolat nélküli áttelepítés).

## <a name="offline-migration"></a>Offline áttelepítés
Offline áttelepítés késései a legegyszerűbb módszere mivel csökkenti a veszélyét annak, hogy a versengés lépett fel. Nem módosítja az adatokat, amíg folyamatban van áthelyezése és szerkezetátalakítás.

Ez a folyamat elméleti szinten, a következő lépéseket tartalmazza:

1. A szilánkok offline megjelölni.
2. Vegyes egyesítéses és az adatok helyezze át az új szilánkok.
3. Ellenőrizze az adatokat.
4. Kapcsolja a hálózatra az új szilánkok.
5. Távolítsa el a régi szilánkcímtárban.

Egyes rendelkezésre állási megőrzéséhez jelölheti meg az 1. lépésben írásvédett eredeti shard ahelyett, így nem érhető el. Ez lehetővé teszi az alkalmazások, az adatok olvasása áthelyezés alatt, de nem módosíthatja.

## <a name="online-migration"></a>Online áttelepítése
Online az áttelepítés akkor összetettebb, de kevésbé zavaró, a felhasználók számára végezhető el, mert az adatok továbbra is elérhető a teljes folyamat során. A folyamat hasonlít a, az offline áttelepítés használatával, azzal a különbséggel, hogy az eredeti shard nincs megjelölve offline (1. lépés). Attól függően, hogy a lépésköz legyen az áttelepítési folyamat (például, hogy elkészült elem vagy shard által shard), az ügyfélalkalmazások számára az adatok hozzáférési kód lehet kezelni a olvasását és írását, amely két helyen használatban van (az eredeti shard és a új shard).

Például egy olyan megoldás, amely támogatja az online áttelepítést, tekintse meg a cikket [méretezés, a rugalmas adatbázis vegyes egyesítéses eszközzel] a Microsoft webhelyén.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták
Amikor kiválasztja az adatok konzisztenciájának bevezetése, a következő minták is lehet a forgatókönyvhöz kapcsolódó:

* A [adatok konzisztencia ismertetése] a Microsoft webhelyén a lap egy elosztott környezetben, például a felhő konzisztencia fenntartása kapcsolatos olyan stratégiák ismerteti.
* A [útmutatást particionálás adatok] oldalon, a Microsoft webhelyén általános áttekintést nyújt a különféle feltételeknek megfelelő elosztott megoldásban partíciók megtervezésére.
* A [horizontális mintát] erről a Microsoft webhelyén foglalja össze a horizontális adatokat néhány gyakori stratégiát.
* A [index táblázat mintát] erről a Microsoft webhelyén azt ábrázolja, hogyan hozhat létre másodlagos indexek adatokat. Egy alkalmazás egy gyűjtemény elsődleges kulcsa nem hivatkozó lekérdezések használatával gyorsan lekérhető adatok ezt a módszert.
* A [materializált nézet mintát] erről a Microsoft webhelyén ismerteti, hogyan lehet adatokat gyors lekérdezési műveletek támogatásához előre megadott nézetek létrehozása céljából. Ez a módszer hasznos lehet a particionált tárolóban Ha összegzett adatok a partíciók több hely különböző pontjain.
* A [használata Azure Content Delivery Network] cikk a Microsoft webhelyén konfigurálásával és az Azure Content Delivery Network használatával további útmutatást nyújt.

## <a name="more-information"></a>További információ
* A lap [Mi az Azure SQL Database?] a Microsoft webhelyén biztosít, amely azt ismerteti, hogyan történő létrehozásáról és használatáról az SQL-adatbázisok részletes dokumentációt.
* A lap [rugalmas adatbázis-szolgáltatások áttekintése] a Microsoft webhelyén egy átfogó bevezetést tartalmaz ahhoz a rugalmas adatbázis.
* A lap [méretezés, a rugalmas adatbázis vegyes egyesítéses eszközzel] a Microsoft webhelyén a vegyes egyesítéses szolgáltatással kezelheti a rugalmas adatbázis szilánkok adatait tartalmazza.
* A lap [az Azure storage méretezhetőségi és Teljesítménycélok](https://msdn.microsoft.com/library/azure/dn249410.aspx) a Microsoft-webhely az aktuális méretezési és az átviteli Sebességkorlát az Azure Storage dokumentumokat.
* A lap [entitás csoport tranzakciók végrehajtása] a Microsoft webhelyén részletes információkat nyújt azokról tranzakciós műveletek végrehajtása az Azure table storage-ban tárolt entitásokat keresztül.
* A cikk [Azure Storage táblázat kialakítási útmutató] a Microsoft webhely adatokat az Azure table storage-ban partícionálásra vonatkozó részletes információkat tartalmaz.
* A lap [használata Azure Content Delivery Network] a Microsoft webhelyén ismerteti, hogyan lehet replikálja az adatokat, amely használatban van az Azure blob Storage tárolóban az Azure Content Delivery Network használatával.
* A lap [Azure Search újdonságai?] a Microsoft webhelyén elérhető lehetőségek érhetők el az Azure Search teljes leírását tartalmazza.
* A lap [szolgáltatási korlátait, az Azure Search] a Microsoft webhelyén a kapacitás az összes Azure Search-példány adatait tartalmazza.
* A lap [a támogatott adattípusokat (Azure Search)] a Microsoft webhely segítségével is kereshető dokumentumok és indexek adattípusokat foglalja össze.
* A lap [Azure Redis Cache] a Microsoft webhelyén Azure Redis Cache bevezetést nyújt.
* A [particionálására: hogyan adatok több Redis-példány között] a Redis-webhelyen lap megvalósításához a particionálás a redis gyorsítótárral kapcsolatos információkat biztosít.
* A lap [a CentOS Linux virtuális gép az Azure-ban futó Redis] a Microsoft webhelyén végigvezeti egy példa, amely azt ismerteti, hogyan hozza létre és konfigurálja az Azure virtuális gépként futó Redis csomópont.
* A [adattípusok] a Redis-webhelyen lap ismerteti, hogy a Redis és az Azure Redis Cache adatok típusát.

[rendelkezésre állását és az Event Hubs következetes]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limit
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Azure Redis Cache]: http://azure.microsoft.com/services/cache/
[Az Azure Storage méretezhetőségi és teljesítménycéloknak]: /azure/storage/storage-scalability-targets
[Az Azure Storage táblázat kialakítási útmutató]: /azure/storage/storage-table-design-guide
[A Polyglot megoldás létrehozása]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/documentdb/documentdb-request-units
[A magas szinten méretezhető megoldások adatelérési: SQL, nosql-alapú és Polyglot adatmegőrzési használatával]: https://msdn.microsoft.com/library/dn271399.aspx
[adatok konzisztencia ismertetése]: http://aka.ms/Data-Consistency-Primer
[Adatok particionálási útmutató]: https://msdn.microsoft.com/library/dn589795.aspx
[Adattípusok]: http://redis.io/topics/data-types
[documentdb-api]: /azure/documentdb/documentdb-introduction
[rugalmas adatbázis-szolgáltatások áttekintése]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[irányelvek és javaslatok az Azure Service Fabric megbízható gyűjtemények]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Index táblázat minta]: http://aka.ms/Index-Table-Pattern
[Materializált nézet minta]: http://aka.ms/Materialized-View-Pattern
[több shard lekérdezése]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Azure Service Fabric áttekintése]: /azure/service-fabric/service-fabric-overview
[A Service Fabric megbízható szolgáltatások partícióazonosító]: /azure/service-fabric/service-fabric-concepts-partitioning
[particionálására: hogyan adatok több Redis-példány között]: http://redis.io/topics/partitioning
[Entitás csoport tranzakciók végrehajtása]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[Redis-fürt oktatóanyag]: http://redis.io/topics/cluster-tutorial
[a CentOS Linux virtuális gép az Azure-ban futó Redis]: http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx
[méretezés, a rugalmas adatbázis vegyes egyesítéses eszközzel]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[használata Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Service Bus kvóták]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[szolgáltatási korlátait, az Azure Search]:  /azure/search/search-limits-quotas-capacity
[horizontális mintát]: http://aka.ms/Sharding-Pattern
[Támogatott adattípusokat (az Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[tranzakciók]: http://redis.io/topics/transactions
[Mi az az Event Hubs?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[Azure Search újdonságai?]: /azure/search/search-what-is-azure-search
[Mi az Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview
