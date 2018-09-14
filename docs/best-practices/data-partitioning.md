---
title: Adatparticionálási útmutató
description: Útmutató a partíciók megfelelő elkülönítéséhez a független felügyelet és hozzáférés érdekében.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: d1d9c1b3cf07f724eb010fc260d86ceb84b789ca
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/13/2018
ms.locfileid: "29059972"
---
# <a name="data-partitioning"></a>Adatparticionálás

Számos nagyobb léptékű megoldásban az adatok egymástól függetlenül felügyelhető és elérhető partíciókba osztva tárolhatók. A particionálási stratégiát gondosan kell megválasztani az előnyök maximalizálása és a negatív hatások minimalizálása érdekében. A particionálás segíthet javítani a skálázhatóságot, csökkenteni a versengést és optimalizálni a teljesítményt. A particionálás egy másik előnye, hogy egy olyan mechanizmust kínál, amellyel az adatok a használati minták mentén oszthatóak fel. Például a régebbi, kevésbé aktív (hideg) adatok olcsóbb adattárolókban archiválhatóak.

## <a name="why-partition-data"></a>Miért kell particionálni az adatokat?
A legtöbb felhőalapú alkalmazás és szolgáltatás a működése során adatokat tárol és kér le. Az alkalmazás által használt adattároló kialakítása jelentős hatással lehet a rendszer teljesítményére, feldolgozási sebességére és skálázhatóságára. A nagy rendszerekben gyakran alkalmazott módszerek egyike az adatok külön partíciókra való felosztása.

> Ebben a cikkben a *particionálás* kifejezés alatt az adatok külön adattárakba való fizikai felosztásának folyamatát értjük. Ez nem ugyanaz, mint az SQL Server táblaparticionálása.

Az adatok particionálása számos előnyt kínál. Például a következők érdekében alkalmazható:

* **A skálázhatóság javítása**. Az önálló adatbázisrendszerek vertikális felskálázásakor a rendszer végül eléri a fizikai hardver korlátait. Ha az adatokat több partícióra osztjuk, amelyek mindegyike külön kiszolgálón üzemel, a rendszer horizontálisan szinte végtelenül felskálázható.
* **A teljesítmény javítása**. Az egyes partíciókon az adatelérési műveletek kisebb mennyiségű adatot érintenek. Amennyiben az adatok megfelelő módon vannak particionálva, a particionálás révén növelhető a rendszer hatékonysága. Az egyszerre több partíciót érintő műveletek párhuzamosan futtathatóak. Az egyes partíciók az azokat használó alkalmazások közelében helyezhetőek el, így minimalizálható a hálózati késés.
* **A rendelkezésre állás javítása**. Az adatok több kiszolgálóra való leosztásával kiküszöbölhető az egypontos meghibásodás kockázata. Ha valamely kiszolgáló működése leáll vagy tervezett karbantartás történik, csak az adott partíció adatai válnak elérhetetlenné. A többi partíción folytatódhatnak a műveletek. A partíciók számának növelésével csökkenthető az egyes kieső kiszolgálók relatív hatása, mivel arányaiban kevesebb adat válik elérhetetlenné. Az egyes partíciók replikálásával tovább csökkenthető az esélye, hogy egyetlen partíció kiesése kihatna a működésre. Emellett lehetővé teszi a folyamatos magas rendelkezésre állást igénylő kritikus adatok elkülönítését az alacsonyabb rendelkezésre állási követelményekkel rendelkező értéktelenebb adatoktól (amilyenek például a naplófájlok).
* **A biztonság javítása**. Az adatok természetétől és particionálásának módjától függően a bizalmas és a nem bizalmas adatok esetleg külön partíciókba, és ezáltal külön kiszolgálókra vagy adattárakba helyezhetőek. A biztonságot ezután hatékonyabban lehet kifejezetten a bizalmas adatokra optimalizálni.
* **A működési rugalmasság megteremtése**. A particionálás rengeteg lehetőséget kínál a működés finomhangolására, a felügyelet hatékonyságának maximalizálására és a költségek minimalizálására. Például különböző stratégiák határozhatók meg a felügyelethez, a monitorozáshoz, a biztonsági mentéshez és helyreállításhoz, valamint egyéb felügyeleti tevékenységekhez az egyes partíciók adatainak fontossága alapján.
* **A használati mintának megfelelő adattárak használata**. A particionálás révén az egyes partíciók különböző típusú adattárakban futtathatóak az ilyen adattárak költségei és beépített szolgáltatásai alapján. Például a nagy mennyiségű bináris adatot blobtárolókban, míg a strukturáltabb adatot dokumentum-adatbázisokban tárolhatja. További információkért lásd a minta- és gyakorlati útmutató [Többnyelvű megoldások létrehozása] foglalkozó szakaszát, valamint a Microsoft webhelyén a [Adathozzáférés nagymértékben skálázható megoldások esetén: az SQL, NoSQL és többnyelvű adatmegőrzés használata] bemutató cikket.

Egyes rendszerek nem valósítják meg a particionálást, mivel inkább költségnek, mint előnynek tekintik. Az ilyen gondolkodásnak általában a következők az okai:

* Számos adattároló rendszer nem támogatja a partíciók közti összekapcsolásokat, és esetenként a hivatkozások integritásának fenntartása is nehézkes lehet egy particionált rendszerben. Gyakran az összekapcsolásokat és az integritás-ellenőrzést az alkalmazáskódban (a particionálási rétegben) kell megvalósítani, ami növelheti az I/O-terhelést és alkalmazás összetettségét.
* A partíciók karbantartása néha nem triviális feladat. Az ideiglenes adatokat tartalmazó rendszerekben időnként újra ki kell egyensúlyozni a partíciókat a versengés és a kritikus pontok elkerülése érdekében.
* Némely általános eszköz nem működik természetesen a particionált adatokkal.

## <a name="designing-partitions"></a>Partíciók tervezése
Az adatok különböző módokon particionálhatóak: vízszintesen, függőlegesen vagy funkcionálisan. A választott stratégia az adatok particionálásának okaitól, valamint az adatokat használó alkalmazások és szolgáltatások követelményeitől függ.

> [!NOTE]
> Az ebben az útmutatóban leírt particionálási sémákat a mögöttes adattárolási módszerektől független módon mutatjuk be. A sémák számos különböző típusú adattárra alkalmazhatóak, beleértve a relációs és NoSQL-adatbázisokat is.
>
>

### <a name="partitioning-strategies"></a>Particionálási stratégiák
A három tipikus adatparticionálási stratégia:

* **Horizontális particionálás** (vagy más néven *horizontális skálázás*). Ebben a stratégiában mindegyik partíció egy önálló, teljes jogú adattár, de mindegyik partíció ugyanazzal a sémával rendelkezik. Az egyes partíciókat *szegmensnek* nevezik, és az adatok egy-egy alhalmazát tárolják, például egy e-kereskedelmi alkalmazásban az ügyfelek adott halmazának megrendeléseit.
* **Vertikális particionálás**. Ebben a stratégiában mindegyik partíció az adattárban tárolt elemek mezőinek egy alhalmazát tartalmazza. A mezők a használati mintáik alapján vannak felosztva. Például a gyakran használt mezők az egyik, a kevésbé gyakran használtak egy másik partícióba kerülhetnek.
* **Funkcionális particionálás**. Ebben a stratégiában az adatok összesítése az alapján történik, hogy a rendszer egyes körülhatárolt kontextusai hogyan használják őket. Egy, a számlázáshoz és a készletkezeléshez külön üzleti funkciókat megvalósító e-kereskedelmi rendszer az egyik partícióban tárolhatja például a számlázási adatokat, egy másikban pedig a készlettel kapcsolatos adatokat.

Fontos megjegyezni, hogy a fenti három stratégia kombinálható is. Az egyik használata nem zárja ki a többit, ezért a particionálási séma kialakításakor érdemes mindhármat számításba venni. Például az adatokat először feloszthatja szegmensekre, majd az egyes szegmensekben lévő adatokat vertikálisan tovább particionálhatja. Ugyanígy egy funkcionális partícióban lévő adatok is szegmensekre bonthatók (majd ezek akár vertikálisan tovább particionálhatóak).

Az egyes stratégiák eltérő követelményei azonban ütközéseket okozhatnak. A rendszer átfogó adatfeldolgozási teljesítménycéljait kiszolgáló particionálási séma kidolgozásához ezeket mindenképp ki kell értékelnie és egyensúlyba kell hoznia. Az alábbi szakaszokban az egyes stratégiákat mutatjuk be részletesebben.

### <a name="horizontal-partitioning-sharding"></a>Horizontális particionálás
Az 1. ábrán a horizontális particionálás áttekintése látható. Ebben a példában a készletadatok termékkulcs alapján vannak szegmensekre bontva. Minden egyes szegmens a szegmenskulcsok betűrend szerint egybefüggő tartományát (A–G és H–Z) tartalmazza.

![Adatok horizontális particionálása partíciókulcs alapján](./images/data-partitioning/DataPartitioning01.png)

*1. ábra Adatok horizontális particionálása partíciókulcs alapján*

A horizontális particionálás segítségével több számítógépre oszthatja le a terhelést, ami csökkenti a versengést és javítja a teljesítményt. A rendszer más kiszolgálókon futó további szegmensek hozzáadásával horizontálisan felskálázható.

Ennek a particionálási stratégiának az alkalmazásakor a legfontosabb tényező a szegmenskulcs megfelelő megválasztása. A rendszer beüzemelését követően a kulcs nehezen módosítható. A kulcsnak biztosítania kell, hogy az adatok particionálása olyan legyen, hogy a terhelés a lehető legegyenletesebb oszoljon meg a szegmensek között.

Az egyes szegmenseknek nem kell hasonló adatmennyiséget tartalmaznia. Sokkal fontosabb a kérelmek számát egyensúlyba hozni. Előfordulhat, hogy egyes szegmensek nagy méretűek, de az egyes elemeikre csak kevés adatelérési művelet irányul. Más szegmensek esetleg kisebbek ugyan, de az egyes elemeket többször érik el. Fontos arról is gondoskodni, hogy az egyes szegmensek ne lépjék túl az adott szegmenst futtató adattár skálázási korlátait (kapacitás és feldolgozási erőforrások tekintetében).

Amennyiben horizontális particionálási sémát alkalmaz, kerülje a kritikus pontok (vagy forró partíciók) kialakítását, amelyek hatással lehetnek a teljesítményre és a rendelkezésre állásra. Ha például a felhasználó nevének első betűje helyett a felhasználóazonosító kivonatát alkalmazza szegmenskulcsként, elkerülheti a gyakoribb és ritkább kezdőbetűk okozta egyenetlen eloszlást. Ez egy tipikus módszer arra, hogy az adatokat egyenletesebben osszák el a partíciók között.

Válasszon olyan szegmenskulcsot, amely révén elkerülhető, hogy később a nagy szegmenseket kisebbekre kelljen felosztani, a kisebbeket nagyobbakká egyesíteni, vagy a partíciók egy halmazában tárolt adatokat leíró sémát módosítani. Az ilyen műveletek nagyon időigényesek lehetnek, és a végrehajtásuk során esetleg néhány szegmenst le is kell állítani.

Ha a szegmensek replikálva vannak, a replikák némelyike esetleg online tartható, amíg a többi felosztása, egyesítése vagy átkonfigurálása folyik. A rendszernek azonban esetleg korlátoznia kell az ezekben a szegmensekben lévő adatokon végrehajtható műveleteket, amíg az átkonfigurálás folyamatban van. Például a replikákon lévő adatok esetleg írásvédetté tehetőek a szegmensek átstrukturálása során esetleg fellépő inkonzisztenciák hatókörének korlátozása végett.

> A fenti szempontokkal kapcsolatos részletesebb információkért és útmutatást, valamint a horizontális particionálást megvalósító adattárak kialakítására vonatkozó ajánlott eljárásokat [Horizontális particionálási minta] ismertető részben talál.
>
>

### <a name="vertical-partitioning"></a>Vertikális particionálás
A vertikális particionálást általában a leggyakrabban elért adatok lekérésével kapcsolatos I/O- és teljesítményköltségek csökkentésére alkalmazzák. A 2. ábrán a vertikális particionálás egy példája látható. Ebben a példában az egyes adatelemek különböző tulajdonságai különböző partíciókon találhatók. Az egyik partíció a gyakrabban elért adatokat, például a termékek nevét, leírását és árát tartalmazza. Egy másik a készletmennyiséget és az utolsó rendelés dátumát tárolja.

![Adatok vertikális particionálása a használati minta alapján](./images/data-partitioning/DataPartitioning02.png)

*2. ábra Adatok vertikális particionálása a használati minta alapján*

Ebben a példában az alkalmazás rendszeresen lekérdezi a termékek nevét, leírását és árát, amikor megjeleníti a termékek részleteit az ügyfelek számára. A készleten lévő mennyiségeket és a gyártótól való utolsó rendelés időpontját egy külön partíció tárolja, mivel ezt a két elemet általában együtt használják.

Ennek a particionálási sémának további előnye, hogy a relatíve ritkábban frissített adatokat (a termékek nevét, leírását és árát) elkülöníti a dinamikusabbaktól (a készleten lévő mennyiségektől és az utolsó megrendelések dátumától). Az alkalmazásnak érdemes lehet a ritkában frissített adatokat gyorsítótáraznia, ha azokat gyakran kell elérni.

Egy másik tipikus forgatókönyv a bizalmas adatok biztonságának növelése. Például ennek megvalósításához a hitelkártyaszámokat és a kártyák kapcsolódó biztonsági hitelesítő számait külön partíciókon tárolják.

A vertikális particionálás használatával csökkenthető továbbá az adatok egyidejű elérésének mennyisége.

> A vertikális particionálás az entitások szintjén működik a tárolóban, részlegesen normalizálja az entitásokat, és *széles* elemekből *keskeny* elemekké bontja le azokat. Kiválóan alkalmazható oszlopalapú, például HBase és Cassandra adattárakhoz. Ha az oszlopok egy gyűjteményében lévő adatok valószínűleg nem változnak majd, megpróbálhat oszloptárolókat is használni az SQL Serverben.
>
>

### <a name="functional-partitioning"></a>Funkcionális particionálás
Az olyan rendszerekben, ahol az alkalmazásban lévő minden egyes külön üzleti területhez vagy szolgáltatáshoz azonosítható egy körülhatárolt kontextus, a funkcionális particionálás megfelelő módszer lehet az elkülönítés és az adatelérési teljesítmény javításához. A funkcionális particionálás egy másik gyakori alkalmazási módja az írható és a jelentési célokat szolgáló írásvédett adatok elkülönítése. A 3. ábrán a funkcionális particionálás áttekintése látható egy példán, ahol a készletadatok elkülönülnek az ügyféladatoktól.

![Adatok funkcionális particionálása körülhatárolt kontextus vagy részterület alapján](./images/data-partitioning/DataPartitioning03.png)

*3. ábra Adatok funkcionális particionálása körülhatárolt kontextus vagy részterület alapján*

Ez a particionálási stratégia csökkentheti az adatelérési versengést a rendszer egyes részei között.

## <a name="designing-partitions-for-scalability"></a>Partíciók tervezése skálázhatóságra
Elengedhetetlen az egyes partíciók méretének és munkaterhelésének mérlegelése és kiegyensúlyozása, hogy az adatok megfelelően legyenek elosztva a maximális skálázhatóság érdekében. Azonban az adatok particionálása során arra is figyelni kell, hogy ne haladják meg az egypartíciós tárolók skálázási korlátait.

A partíciók skálázhatóságra tervezése során kövesse az alábbi lépéseket:

1. Az alkalmazás elemzésével ismerje meg a hozzáférési mintákat, például az egyes lekérdezések által visszaadott eredményhalmazok méretét, az adatelérési gyakoriságot, az eredendő késést és a kiszolgálóoldali számítási feldolgozás követelményeit. Sok esetben néhány nagyobb entitás igényli a szükséges erőforrások többségét.
2. Az elemzés segítségével határozza meg az aktuális és a jövőbeli skálázhatósági célokat, például az adatok méretét és a munkaterhelést. Ezután ossza el az adatokat a partíciók közt a skálázhatósági célok teljesítéséhez. A horizontális particionálási stratégiában az egyenletes eloszláshoz fontos a megfelelő szegmenskulcs megválasztása. További információkért lásd [Horizontális particionálási minta].
3. Gondoskodjon róla, hogy az egyes partíciók erőforrásai elegendőek legyenek a skálázhatósági követelmények kezelésére az adatméret és a feldolgozási sebesség tekintetében. Például az egyes partíciókat futtató csomópontok korlátozhatják az általuk biztosított tárterület mennyiségét, a feldolgozási teljesítményt vagy a hálózati sávszélességet. Ha az adattárolási és feldolgozási követelmények valószínűleg meghaladják majd ezeket a korlátokat, szükség lehet a particionálási stratégia finomítására vagy az adatok további felosztására. Például egy lehetséges skálázhatósági megközelítés a naplózási adatok elkülönítése az alkalmazás alapszolgáltatásaitól. Ez az adattárak elkülönítésével oldható meg, hogy a teljes adattárolási igény ne haladja meg a csomópont skálázási korlátait. Ha az adattárak teljes száma meghaladja a csomópont korlátait, szükséges lehet további tárolócsomópontokat üzembe helyezni.
4. Monitorozza a rendszert az éles üzem során, és győződjön meg róla, hogy az adatok a várakozásnak megfelelően vannak elosztva, és hogy a partíciók képesek kezelni a rájuk osztott terhelést. Lehetséges, hogy a tényleges használat nem egyezik az elemzés alapján várható használattal. Ebben az esetben újra kiegyensúlyozhatók a partíciók. Ennek hiányában esetleg át kell alakítani a rendszer bizonyos részeit, hogy elérhető legyen a szükséges egyensúly.

Vegye figyelembe, hogy egyes felhőkörnyezetek az infrastruktúra határai mentén foglalják le az erőforrásokat. Bizonyosodjon meg róla, hogy a kiválasztott határ korlátai elegendő teret biztosítanak az adatmennyiség várható növekedéséhez az adattárolás, a feldolgozási teljesítmény és a sávszélesség tekintetében.

Például Azure-táblatárolók használata esetén a nagy terhelésű szegmensek több erőforrást igényelhetnek, mint amennyi egy adott partíció számára elérhető a kérelmek feldolgozásához. (Az egy partíció által egy adott időszakban feldolgozható kérelmek mennyisége korlátozva van. További információkat [Az Azure Storage skálázhatósági és teljesítménycéljai] bemutató oldalon talál a Microsoft webhelyén.)

 Amennyiben valóban ez a helyzet, a szegmenst esetleg újra kell particionálni a terhelés szétosztása érdekében. Ha a táblák teljes mérete vagy feldolgozási sebessége meghaladja a tárfiók kapacitását, szükséges lehet további tárfiókokat létrehozni, és elosztani a táblákat a fiókok közt. Ha a tárfiókok száma meghaladja az egy előfizetésben elérhető fiókok számát, több előfizetésre lehet szükség.

## <a name="designing-partitions-for-query-performance"></a>Partíciók tervezése lekérdezési teljesítményre
A lekérdezési teljesítmény gyakran kisebb adatkészletek használatával és a lekérdezések párhuzamos futtatásával növelhető. Mindegyik partíciónak a teljes adatkészlet egy kis részét kell tartalmaznia. A mennyiség csökkenésével javulhat a lekérdezések teljesítménye. A particionálás azonban nem váltja ki az adatbázis megfelelő kialakítását és konfigurálását. Például relációs adatbázisok használata esetén mindenképp gondoskodnia kell róla, hogy az tartalmazza a szükséges indexeket.

A partíciók lekérdezési teljesítményre tervezése során kövesse az alábbi lépéseket:

1. Vizsgálja meg az alkalmazás követelményeit és teljesítményét:
   * Az üzleti követelmények alapján határozza meg a kritikus lekérdezéseket, amelyeknek mindig gyorsan kell lefutnia.
   * A rendszer monitorozásával azonosítsa a mindig lassan lefutó lekérdezéseket.
   * Határozza meg, hogy melyik lekérdezések futnak a leggyakrabban. Az egyes lekérdezés egyes példányai önmagukban talán minimális költséggel járnak, az összesített erőforrás-használatuk azonban jelentős lehet. Esetleg érdemes lehet az ilyen lekérdezések által beolvasott adatokat leválasztani egy külön partícióba, vagy akár egy gyorsítótárba.
2. Particionálja a teljesítményt csökkentő adatokat:
   * Korlátozza az egyes partíciók méretét, hogy a lekérdezések válaszideje a célon belül maradjon.
   * A szegmenskulcsot úgy alakítsa ki, hogy az alkalmazás könnyen megtalálja a partíciót, ha horizontális particionálást alkalmaz. Így a lekérdezésnek nem kell az összes partíciót átvizsgálnia.
   * Mérlegelje a partíciók elhelyezését. Ha lehetséges, próbálja az adatokat olyan partíciókban tárolni, amelyek földrajzilag közel helyezkednek el az adatokhoz hozzáférő alkalmazásokhoz és felhasználókhoz.
3. Ha egy entitás a feldolgozási sebességre vagy lekérdezési teljesítményre vonatkozó követelményekkel rendelkezik, alkalmazzon funkcionális particionálást az adott entitás alapján. Ha ez még mindig nem elégíti ki a követelményeket, alkalmazzon horizontális particionálást is. A legtöbb esetben egyetlen particionálási stratégia elegendő lesz, egyes esetekben azonban a két stratégia kombinálása hatékonyabbnak bizonyul.
4. Vegye fontolóra aszinkron lekérdezések párhuzamos futtatását a partíciókon a teljesítmény javítása érdekében.

## <a name="designing-partitions-for-availability"></a>Partíciók tervezése rendelkezésre állásra
Az adatok particionálásával javítható az alkalmazások rendelkezésre állása, mivel így a teljes adatkészlet nem jelent egypontos meghibásodási helyet, és az adatkészlet egyes részhalmazai egymástól függetlenül kezelhetők. A kritikus adatokat tartalmazó partíciók replikálása szintén javíthatja a rendelkezésre állást.

A partíciók kialakítása és megvalósítása során vegye figyelembe a rendelkezésre állást befolyásoló alábbi tényezőket:

* **Mennyire kritikusak az adatok az üzleti tevékenységek szempontjából**. Bizonyos adatok kritikus üzleti információkat tartalmazhatnak, például számlázási adatokat vagy banki tranzakciók adatait. Más adatok kevésbé fontos működési adatok, például naplófájlok, teljesítmény-nyomkövetések és hasonlók lehetnek. Az egyes adattípusok azonosítását követően mérlegelje a következőket:
  * A kritikus fontosságú adatokat tárolja magas rendelkezésre állású partíciókon, megfelelő biztonsági mentési tervvel.
  * Hozzon létre külön felügyeleti és monitorozási mechanizmusokat vagy eljárásokat az egyes adatkészletek különböző kritikusságú elemeihez. Az ugyanolyan kritikusságú adatokat helyezze ugyanabba a partícióba, hogy megfelelő gyakorisággal egyszerre készíthessen biztonsági másolatot róluk. Például a banki tranzakciók adatait tároló partíciókról valószínűleg gyakrabban kell biztonsági másolatot készíteni, mint a naplózási és nyomkövetési információkat tartalmazóakról.
* **Hogyan felügyelhetőek az egyes partíciók**. A partíciók a független felügyeletet és karbantartást biztosító kialakítása több előnnyel is jár. Példa:
  * Ha valamelyik partíció leáll, önállóan helyreállítható, anélkül, hogy befolyásolná a többi partíció adatait használó alkalmazáspéldányok működését.
  * Az adatok földrajzi területek szerinti particionálásával az ütemezett karbantartás minden helyen a csúcsidőn kívül végezhető el. Gondoskodjon róla, hogy a partíciók ne legyenek túl nagyok a tervezett karbantartás adott időablakban való elvégzéséhez.
* **Érdemes-e a kritikus fontosságú adatokat a partíciók közt replikálni**. Ez a stratégia javíthatja a rendelkezésre állást és a teljesítményt, bár konzisztenciaproblémákat okozhat. Bizonyos időt igénybe vesz, amíg az egyes partíciók adatain végzett módosítások szinkronizálódnak az összes replikára. Ez alatt az idő alatt az adatértékek a különböző partíciókban különbözőek lesznek.

## <a name="understanding-how-partitioning-affects-design-and-development"></a>A particionálás tervezési és fejlesztési szempontjai
A particionálás használata bonyolultabbá teszi a rendszer tervezését és fejlesztését. A particionálást a rendszer alapvető részeként kell tekinteni a tervezés során, még ha a rendszer eleinte csak egyetlen partícióval rendelkezik is. Ha a particionálást csak utólag kezdi tervezni, amikor a rendszer már teljesítmény- és skálázhatósági problémákkal küzd, a feladat sokkal bonyolultabb lesz, mivel a már működő rendszert karban kell tartani.

Amikor ebben a környezetben a particionálás bevezetéséhez frissíti a rendszert, módosítania kell az adathozzáférési logikát. Emellett a partíciók közötti elosztáshoz nagy mennyiségű meglévő adatot kell migrálni, gyakran úgy, hogy a felhasználók továbbra is elvárják a rendszer megfelelő működését.

Esetenként a particionálást nem tekintik fontosnak, mivel kezdetben még kisméretű az adatkészlet, és egy kiszolgáló könnyen képes kezelni azt. Ez lehet például a helyzet egy olyan rendszer esetében, amelynek a méretét nem tervezik növelni az induló állapothoz képest, a legtöbb kereskedelmi rendszernek azonban mégis növekednie kell a felhasználók számának növekedésével. Az ilyen növekedés általában az adatok mennyiségének növekedésével is jár.

Azt is fontos megérteni, hogy a particionálás nem mindig a nagyméretű adattárak függvénye. Például egy kisméretű adattárnak is lehet intenzív forgalma több száz egyidejű ügyfél irányában. Ilyen esetben az adatok particionálása segíthet csökkenteni a versengést és növelni a teljesítményt.

Az adatparticionálási sémák kialakításakor vegye figyelembe a következő szempontokat:

* **Amikor csak lehetséges, a leggyakrabban használt adatbázis-műveletekben érintett adatokat tartsa együtt az egyes partíciókban a több partíciót érintő adatelérési műveletek minimalizálása érdekében**. A több partícióra kiterjedő lekérdezések több időt vehetnek igénybe, mint az egyetlen partíciót érintőek, azonban a partíciók a lekérdezések egy adott halmazra való optimalizálása kedvezőtlenül befolyásolhatja a többi lekérdezéshalmazt. Ha a több partícióra kiterjedő lekérdezések nem kerülhetők el, a lekérdezések ideje a lekérdezések párhuzamos futtatásával és az eredmények az alkalmazásban való összegzésével minimalizálható. Ez a módszer bizonyos esetekben nem alkalmazható, például ha egy lekérdezés eredményét a következő lekérdezésben kell használni.
* **Ha a lekérdezések viszonylag statikus referenciaadatokra vonatkoznak, például az irányítószámok táblázatára vagy terméklistákra, ezeket az adatokat érdemes lehet az összes partícióra replikálni, hogy ne kelljen annyi külön keresési műveletet végezni a különböző partíciókban**. Ez a megközelítés annak a valószínűségét is csökkentheti, hogy a referenciaadatok olyan „forró” adatkészletté váljanak, amelyre a rendszer egészéből intenzív forgalom irányul. Az ilyen referenciaadatok esetleges szinkronizálása azonban további költségekkel járhat.
* **Ahol csak lehetséges, a vertikális és funkcionális partíciókban minimalizálja a hivatkozásintegritás-igényeket**. Ezekben a sémákban az alkalmazás feladata a hivatkozásintegritás fenntartása a különböző partíciók között az adatok frissítése és használata során. Az olyan lekérdezések, amelyeknek több partíció adatait kell összekapcsolni, lassabban futnak az egy partíció adatait összekapcsolóknál, mivel az alkalmazásnak általában egymást követő lekérdezéseket kell végrehajtania, egyiket a másik után, különböző idegen kulcsokkal. Ehelyett érdemes replikálni a vonatkozó adatokat vagy megszüntetni azok normalizáltságát. A partíciók közti összekapcsolást nem igénylő lekérdezések idejének minimalizálásához futtassa párhuzamosan a lekérdezéseket a külön partíciókban, és az adatokat az alkalmazásban kapcsolja össze.
* **Mérlegelje, hogy a particionálási séma milyen hatással lehet az adatok a partíciók közti konzisztenciájára.** Mérje fel, hogy az erős konzisztencia valóban követelmény-e. Ehelyett egy, a felhőben gyakran alkalmazott megközelítés a végső konzisztencia megvalósítása. Az egyes partíciók adatainak frissítése külön történik, és az alkalmazáslogika biztosítja, hogy a frissítések mind sikeresen mennek végbe. A logika emellett a végül konzisztens műveletek keretében futtatott adatlekérdezésekből eredő esetleges inkonzisztenciákat is kezeli. A végső konzisztenciával kapcsolatos további információkat az [Adatkonzisztencia – Ismertető] szakaszban találja.
* **Mérlegelje, hogyan találják meg a lekérdezések a megfelelő partíciót**. Ha egy lekérdezésnek az összes partíciót át kell vizsgálnia a szükséges adatok megtalálásához, az jelentős mértékben kihathat a teljesítményre, még akkor is, ha több párhuzamos lekérdezés fut. A vertikális és a funkcionális particionálási stratégiákkal használt lekérdezések természetes módon képesek meghatározni a partíciókat. A horizontális particionálás esetében azonban az egyes elemek helyének megkeresése nehézkes lehet, mert minden szegmens ugyanazt a sémát használja. Egy tipikus megoldás erre a horizontálisan particionált rendszerekben egy olyan térkép fenntartása, amelynek használatával meg lehet határozni az egyes adatelemek helyét a szegmensek közt. Ez a térkép az alkalmazás horizontális particionálási logikájában valósítható meg, vagy az adattár is karbantarthatja, ha támogatja a transzparens horizontális particionálást.
* **Horizontális particionálási stratégia alkalmazása esetén mérlegelje a szegmensek rendszeres kiegyensúlyozásának szükségességét**. Ez segíti az adatok egyenletes elosztását méret és munkaterhelés alapján a kritikus pontok elkerülése, a lekérdezési teljesítmény maximalizálása és a fizikai tárhelykorlátok megkerülése érdekében. Ez azonban egy összetett feladat, amelyhez általában egy egyéni eszköz vagy folyamat használata szükséges.
* **Ha mindegyik partíciót replikálja, ez további védelmet biztosít a meghibásodások ellen**. Ha valamelyik replika leáll, a lekérdezések egy másik, működő példányra irányíthatóak.
* **Ha eléri valamely particionálási stratégia fizikai korlátait, érdemes lehet a skálázhatóságot egy másik szintre kiterjeszteni**. Például ha a particionálás az adatbázis szintjén valósul meg, a partíciókat esetleg több adatbázisban szükséges elhelyezni vagy replikálni. Ha a particionálás már eleve az adatbázis szintjén van megvalósítva, és a fizikai korlátok problémát jelentenek, ez jelezheti azt, hogy a partíciókat esetleg több üzemeltetési fiókban szükséges elhelyezni vagy replikálni.
* **Kerülje a több partícióban lévő adatokat használó tranzakciókat**. Egyes adattárak tranzakció-konzisztenciát és integritást valósítanak meg az adatokat módosító műveletekre, de csak abban az esetben, ha az adatok egyetlen partíción találhatóak. Ha több partícióra kiterjedő tranzakciótámogatás szükséges, ezt valószínűleg az alkalmazáslogika részeként kell megvalósítania, mivel a legtöbb particionálási rendszer ezt natív módon nem támogatja.

Minden adattárhoz bizonyos üzemeltetési felügyeleti és monitorozási tevékenységeket is végezni kell. Ilyen feladat lehet az adatok betöltése, az adatok biztonsági mentése és helyreállítása, az adatok átrendezése, valamint a rendszer megfelelő és hatékony működésének biztosítása.

Mérlegelje az üzemeltetési felügyeletet befolyásoló alábbi tényezőket:

* **Hogyan valósíthatóak meg a megfelelő felügyeleti és üzemeltetési feladatok az adatok particionálása esetén**. Ilyen feladatok lehetnek a biztonsági mentés és visszaállítás, az adatok archiválása, a rendszer monitorozása, valamint egyéb felügyeleti feladatok. Például a logikai konzisztencia fenntartása a biztonsági mentési és visszaállítási műveletek során kihívást jelenthet.
* **Hogyan tölthetőek be az adatok több partícióba, és hogyan adhatók hozzá a külső forrásokból érkező új adatok**. Egyes eszközök és segédprogramok esetleg nem támogatják a horizontálisan particionált adatműveleteket, például az adatok a megfelelő partícióba való betöltését. Ez azt jelenti, hogy esetleg új eszközöket és segédprogramokat kell fejlesztenie vagy beszereznie.
* **Hogyan oldható meg az adatok rendszeres archiválása és törlése**. A partíciók méretének túlzott növekedését elkerülendő az adatokat rendszeresen (például havonta) archiválni és törölni kell. Szükség lehet az adatok egy másik archiválási sémának megfelelő átalakítására.
* **Hogyan találhatók meg az adatintegritási problémák**. Vegye fontolóra egy rendszeres folyamat futtatását az adatintegritási problémák felderítésére, például ha egy adott partíció adatai egy másik partíció hiányzó adataira hivatkoznak. A folyamat megpróbálhatja automatikusan javítani ezeket a problémákat, vagy riaszthatja a kezelőt, aki manuálisan oldhatja meg a problémát. Például egy e-kereskedelmi alkalmazásban a megrendelések információit tárolhatja az egyik partíció, míg a megrendelésekben szereplő sortételeket egy másik. A megrendelések feladását végző folyamatnak így más partíciókhoz kell adatokat hozzáadni. Ha a folyamat hibázik, egyes tárolt sortételekhez esetleg nem tartozik majd megfelelő megrendelés.

A különféle adattárolási technológiák általában saját funkciókat biztosítanak a particionálás támogatására. A következő szakaszok az Azure-alkalmazások által általánosságban használt adattárakban megvalósított lehetőségeket összegzik. Ezen felül leírnak különböző szempontokat is az ezeket a lehetőségeket optimálisan kiaknázó alkalmazások tervezéséhez.

## <a name="partitioning-strategies-for-azure-sql-database"></a>Az Azure SQL Database particionálási stratégiái
Az Azure SQL Database egy felhőben futó, szolgáltatásként nyújtott relációs adatbázis. Az alapját a Microsoft SQL Server képezi. A relációs adatbázis az adatokat táblákba osztja le, és mindegyik tábla sorok sorozataként tárolja az entitásokkal kapcsolatos információkat. Az egyes sorok oszlopai tárolják az adott entitás egyes mezőiben lévő adatokat. A Microsoft webhely [Mi az Azure SQL Database?] című oldala részletes dokumentációval szolgál az SQL-adatbázisok létrehozásához és használatához.

## <a name="horizontal-partitioning-with-elastic-database"></a>Horizontális particionálás az Elastic Database-zel
Az egyes SQL-adatbázisokban tárolható adatok mennyisége korlátozott. A feldolgozási sebességet architekturális tényezők és az adatbázis által támogatott egyidejű kapcsolatok száma korlátozzák. A SQL Database Elastic Database szolgáltatása támogatja az SQL-adatbázisok horizontális skálázását. Az Elastic Database használatával az adatokat több SQL-adatbázisra elosztott szegmensekre particionálhatja. A kezelendő adatok mennyiségének növekedésével vagy csökkenésével hozzá is adhat és el is távolíthat szegmenseket. Az Elastic Database szolgáltatás emellett segít csökkenteni a versengést is, mivel elosztja a terhelést az adatbázisok között.

> [!NOTE]
> Az Elastic Database az Azure SQL Database Federation szolgáltatásait váltotta le. A meglévő SQL Database Federation telepítések az összevonás-migrálási segédprogrammal migrálhatóak az Elastic Database-re. Kialakíthat egy saját horizontális particionálási mechanizmust is, amennyiben a megvalósítani kívánt forgatókönyv nem adja magát természetesen az Elastic Database által nyújtott szolgáltatásokhoz.
>
>

Minden egyes szegmens egy SQL-adatbázisként van megvalósítva. Az egyes szegmensek több adatkészletet (a továbbiakban *shardletet*) is tartalmazhatnak. Minden egyes adatbázis maga biztosítja a benne található shardleteket leíró metaadatokat. A shardlet lehet egyetlen adatelem, vagy az elemek egy csoportja, amelyek mind ugyanazzal a shardletkulccsal rendelkeznek. Például ha egy több-bérlős alkalmazás adatainak horizontális particionálását végzi, a shardletkulcs lehet a bérlő azonosítója, és egy adott bérlő összes adatát ugyanaz a shardlet tárolhatja. A többi bérlő adatai különböző shardletekben lehetnek.

A programozói feladata, hogy az egyes adatkészletekhez shardletkulcsokat rendeljen. Egy külön SQL-adatbázis szolgál globális szegmenstérkép-kezelőként. Ez az adatbázis a rendszerben lévő összes szegmens és shardlet listáját tartalmazza. Az adatokat elérő ügyfélalkalmazások először a globális szegmenstérkép-kezelő adatbázishoz kapcsolódnak, ahonnan lekérik, majd helyben gyorsítótárazzák a szegmenstérképet (amely a szegmenseket és shardleteket sorolja fel).

Az alkalmazás ezután az információk alapján irányítja a megfelelő szegmensre az adatkérelmet. Ezek a funkciók egy sor API mögé vannak bújtatva, amelyeket az Azure SQL Database NuGet-csomagként elérhető Elastic Database-ügyfélkódtára tartalmaz. A Microsoft webhely [Az Elastic Database szolgáltatásainak áttekintése] oldala átfogóbban ismerteti az Elastic Database szolgáltatást.

> [!NOTE]
> A globális szegmenstérkép-kezelő adatbázis replikálásával csökkenthető a késés, és javítható a rendelkezésre állás. Ha az adatbázist valamelyik prémium tarifacsomag használatával valósítja meg, az aktív georeplikáció megfelelő konfigurálásával folyamatosan replikálhatja az adatokat különböző régiókban lévő adatbázisokba. Minden olyan régióban hozzon létre egy másolatot az adatbázisról, ahol találhatók felhasználók. Ezután konfigurálja az alkalmazást, hogy ehhez a másolathoz kapcsolódjon a szegmenstérkép lekéréséhez.
>
> Egy másik módszerrel a szegmenstérkép-kezelő adatbázis az Azure SQL Data Sync vagy egy Azure Data Factory-folyamat használatával is replikálható a régiók közt. Az ilyen típusú replikáció időközönként fut csak, és az olyan esetekben megfelelő, ha a szegmenstérkép ritkán módosul. Megjegyzendő, hogy a szegmenstérkép-kezelő adatbázist nem szükséges prémium tarifacsomag használatával hozni létre.
>
>

Az Elastic Database két sémát kínál az adatok shardletekre való leképezésére és a szegmenseken való eltárolására:

* A **listás szegmenstérkép** egyetlen kulcs és egy shardlet közti társításokat ír le. Például egy több-bérlős rendszerben az egyes bérlők adatai egy egyedi kulccsal lehetnek társítva, majd egy külön shardletben tárolhatók. Az adatvédelem és az elkülönítés érdekében (ez utóbbi megakadályozza, hogy az egyes bérlők kimerítsék a többi bérlő számára elérhető adattárolási erőforrásokat) az egyes shardletek tárolhatóak külön szegmensekben is.

![Bérlők adatainak tárolása külön szegmensekben listás szegmenstérkép használatával](./images/data-partitioning/PointShardlet.png)

*4. ábra Bérlők adatainak tárolása külön szegmensekben listás szegmenstérkép használatával*

* A **tartomány-szegmenstérkép** az összefüggő tartományba eső értékű kulcsok és egy shardlet közti társításokat írja le. Az előzőekben leírt több-bérlős példában a dedikált shardletek megvalósításának alternatívájaként bérlők egy csoportjának (mindegyik saját kulccsal rendelkezik) adatait csoportosíthatja ugyanabba a shardletbe. Ez a séma kevésbé költséges, mint az első (mivel a bérlők osztoznak az adattárolási erőforrásokon), de fennáll a kockázata az adatvédelem és az elkülönítés sérülésének.

![Bérlőtartomány adatainak tárolása tartomány-szegmenstérkép használatával](./images/data-partitioning/RangeShardlet.png)

*5. ábra Bérlőtartomány adatainak tárolása tartomány-szegmenstérkép használatával*

Ne feledje, hogy egyetlen szegmens több shardlet adatait is tartalmazhatja. Például listás shardletek használatával több nem egymást követő shardlet adatait is tárolhatja egyetlen szegmensben. A tartományshardleteket és a listás shardleteket keverheti is egyazon szegmensen belül, bár ekkor külön térképeken szerepelnek majd a globális szegmenstérkép-kezelő adatbázisban. (A globális szegmenstérkép-kezelő adatbázis több szegmenstérképet is tartalmazhat.) A 6. ábra ezt a megközelítést mutatja be.

![Több szegmenstérkép megvalósítása](./images/data-partitioning/MultipleShardMaps.png)

*6. ábra Több szegmenstérkép megvalósítása*

A megvalósított particionálási séma jelentős hatással lehet a rendszer teljesítményére. Emellett befolyásolhatja a szegmensek hozzáadásának és eltávolításának, vagy az adatok az egyes szegmensekre való particionálásának szükséges sebességét is. Ha az adatokat az Elastic Database-zel tervezi particionálni, vegye figyelembe a következő szempontokat:

* Az együtt használt adatokat csoportosítsa egy szegmensre, és kerülje az olyan műveleteket, amelyeknek több szegmenset is el kell érniük. Ne feledje, hogy az Elastic Database használata esetén mindegyik szegmens egy önálló SQL-adatbázis, és az Azure SQL Database nem támogatja az adatbázisok közti összekapcsolásokat (amelyeket az ügyféloldalon kell végrehajtani). Azt se feledje, hogy az Azure SQL Database-ben egyik adatbázis hivatkozásintegritási megkötései, triggerei és tárolt eljárásai nem hivatkozhatnak egy másik objektumaira. Ezért kerülje az olyan rendszerek kialakítását, amelyek szegmensek közti függőségeket tartalmaznak. Az SQL-adatbázisok azonban tartalmazhatnak olyan táblákat, amelyek a lekérdezések és egyéb műveletek által gyakran használt referenciaadatok másolatát tárolják. Ezeknek a tábláknak nem szükséges egyik adott shardlethez sem tartoznia. Az ilyen adatok szegmensek közötti replikálásával elkerülhető, hogy több adatbázis adatait kelljen összekapcsolni. Ideális esetben az ilyen adatoknak statikusnak vagy ritkán frissítettnek kell lenniük, hogy minimalizálható legyen a replikációigény és az adatok elévülésének esélye.

  > [!NOTE]
  > Bár az SQL Database nem támogatja az adatbázisok közti összekapcsolásokat, szegmensek közötti lekérdezéseket végrehajthat az Elastic Database API-val. Ezek a lekérdezések transzparens módon iterálhatóak a szegmenstérképek által hivatkozott összes shardlet adatain. Az Elastic Database API a szegmensek közti lekérdezéseket külön lekérdezések sorozatára bontja (adatbázisonként egy lekérdezésre), majd egyesíti az eredményeket. További információkat [Többszegmenses lekérdezés] foglalkozó oldalon talál a Microsoft webhelyén.
  >
  >
* Az ugyanahhoz a szegmenstérképhez tartozó shardletekben tárolt adatoknak ugyanazzal a sémával kell rendelkezniük. Például ne hozzon létre olyan listás szegmenstérképet, amely egyfelől bérlőadatokat tartalmazó szegmensekre, másfelől termékinformációkat tartalmazó szegmensekre is mutat. Ezt a szabályt az Elastic Database nem tartatja be, azonban az adatok kezelése és lekérdezése rendkívül bonyolulttá válik, ha az egyes shardletek különböző sémákkal rendelkeznek. Az iménti példában jó megoldás lehet, ha két listás szegmenstérképet hoz létre: egy olyat, amely a bérlőadatokra, és egy másikat, amely a termékinformációkra mutat. Ne feledje, hogy a különböző shardleteken tárolt adatok tárolhatók ugyanabban a szegmensben.

  > [!NOTE]
  > Az Elastic Database API szegmensek közötti lekérdezési funkcionalitásának alapja, hogy a szegmenstérképen szereplő mindegyik shardlet ugyanazzal a sémával rendelkezik.
  >
  >
* A tranzakciós műveletek csak egyazon szegmensen belül tárolt adatokon működnek, szegmensek között nem. A tranzakciók több shardletet is érinthetnek, amennyiben azok egyazon szegmens részét képezik. Így ha az üzleti logikának kell tranzakciókat végrehajtania, az érintett adatokat tárolja ugyanazon a szegmensen, vagy valósítson meg végső konzisztenciát. További információkat az [Adatkonzisztencia – Ismertető] szakaszban talál.
* A szegmenseket tárolja az azokban lévő adatokat használó felhasználók közelében (azaz ügyeljen a szegmensek megfelelő földrajzi elhelyezésére). Ezzel a stratégiával csökkenthetők a késések.
* Figyeljen arra, hogy ne legyenek egyszerre a rendszerben rendkívül aktív (kritikus pont) és viszonylag inaktív szegmensek is. Próbálja a terhelést egyenletesen elosztani a szegmensek között. Ehhez kivonatalapú szegmenskulcsokra lehet szükség.
* Ha a szegmensek földrajzi elhelyezkedése lényeges, gondoskodjon róla, hogy a kivonatalapú kulcsok az adatokat használó felhasználókhoz közeli szegmensekben tárolt shardletekre mutatnak.
* Jelenleg az SQL-adattípusok korlátozott halmaza használható csak shardletkulcsként: az *int, a bigint, a varbinary* és a *uniqueidentifier*. Az SQL *int* és *bigint* típusai a C# *int* és *long* adattípusainak felelnek meg, és ugyanazokkal a tartományokkal rendelkeznek. Az SQL *varbinary* típusa a C#-ban *Byte*-tömbökkel kezelhető, az SQL *uniqueidentier* típusa pedig a .NET-keretrendszer *Guid* osztályának felel meg.

Ahogy a neve is mutatja, az Elastic Database (azaz rugalmas adatbázis) használatával az adatok mennyiségének növekedésének vagy csökkenésének megfelelően adhatók hozzá vagy távolíthatók el a szegmensek. Az Azure SQL Database Elastic Database ügyfélkódtár API-jaival az alkalmazások dinamikusan hozhatnak létre és törölhetnek szegmenseket (és transzparens módon frissíthetik a szegmenstérkép-kezelőt). A szegmensek eltávolítása azonban egy destruktív művelet, amelyet követően a szegmensben lévő összes adatot törölni kell.

Ha egy alkalmazásban fel kell osztani vagy egyesíteni kell a szegmenseket, az Elastic Database ehhez egy külön felosztási-egyesítési szolgáltatást kínál. Ez a szolgáltatás egy felhőben üzemeltetett szolgáltatáson fut (ezt a fejlesztőnek kell létrehoznia), és biztonságosan migrálja az adatokat a szegmensek közt. További információkat [Skálázás az Elastic Database felosztási-egyesítési eszközének használatával] témában talál a Microsoft webhelyén.

## <a name="partitioning-strategies-for-azure-storage"></a>Az Azure Storage particionálási stratégiái
Az Azure Storage négy absztrakt entitást kínál az adatok kezeléséhez:

* A Blob Storage a strukturálatlan objektumadatokat tárolja. Egy blob állhat bármilyen szövegből vagy bináris adatból, lehet például egy dokumentum, egy médiafájl vagy egy alkalmazástelepítő. A Blob Storage más néven Objektumtárnak is hívható.
* A Table Storage a strukturált adatkészleteket tárolja. A Table Storage a NoSQL-kulcsattribútumok adattára, amely gyors fejlesztési lehetőségeket és nagy adatmennyiségek gyors elérését biztosítja.
* A Queue Storage megbízható üzenetküldést biztosít a munkafolyamat-feldolgozáshoz és a felhőszolgáltatás összetevői közötti kommunikációhoz.
* A File Storage közös tárterületet biztosít a szabványos SMB-protokollt használó örökölt alkalmazások számára. Az Azure virtuális gépek és a felhőszolgáltatás csatlakoztatott megosztásokon keresztül adatokat oszthatnak meg az alkalmazások összetevői között, a helyszíni alkalmazások pedig a fájlszolgáltatás REST API-ján keresztül hozzáférhetnek a megosztott fájladatokhoz.

A táblatárolók és a blobtárolók lényegében kulcs-érték tárolók. A táblatárolók strukturált, a blobtárolók pedig strukturálatlan adatok tárolására vannak optimalizálva. A tárolók üzenetsorai egy megfelelő mechanizmust biztosítanak a lazán kapcsolt, skálázható alkalmazások létrehozásához. A tábla-, fájl- és blobtárolók, valamint a tároló-üzenetsorok az Azure tárfiókok környezetében jönnek létre. A tárfiókok a redundancia három formáját támogatják:

* A **helyileg redundáns tárolás** az adatokat három példányban tárolja egyetlen adatközponton belül. Az ilyen típusú redundancia a hardverhibákkal szemben védelmet nyújt, az olyan katasztrófák esetén azonban nem, amelyek az egész adatközpontot érintik.
* A **zónaredundáns tárolás** az adatokat három példányban tárolja különböző adatközpontokban ugyanabban a régióban (vagy két földrajzilag egymás közelében lévő régióban). Az ilyen típusú redundancia védelmet nyújt a teljes adatközpontot érintő katasztrófákkal szemben, egy teljes régiót érintő, nagy hálózati kimaradások esetén azonban nem. Megjegyzendő, hogy a zónaredundáns tárolás jelenleg csak blokkblobok esetén érhető el.
* A **georedundáns tárolás** az adatokat hat példányban tárolja: három példányt egy adott régióban (a helyi régióban), és másik három példányt egy távoli régióban. Az ilyen típusú redundancia a legmagasabb szintű katasztrófavédelmet biztosítja.

A Microsoft közzétett skálázhatósági célokat az Azure Storage-re vonatkozóan. További információkat [Az Azure Storage skálázhatósági és teljesítménycéljai] bemutató oldalon talál a Microsoft webhelyén. A teljes tárfiók-kapacitás jelenleg legfeljebb 500 TB lehet. (Ez a tábla-, fiók- és blobtárolókban tárolt adatokat, valamint a tároló-üzenetsorokban található nem feldolgozott üzeneteket foglalja magában.)

A tárfiókok maximális kérelemsebessége (1 KB-os entitás-, blob- vagy üzenetméretet feltételezve) másodpercenként 20 000 kérelem. A tárfiók maximális IOPS-korlátja fájlmegosztásonként legfeljebb 1000 (8 KB-os mérettel). Ha a rendszer várhatóan meghaladja majd ezeket a korlátokat, vegye fontolóra a terhelés elosztását több tárfiókra. Egyetlen Azure-előfizetés alatt legfeljebb 200 tárfiók hozható létre. Ezek a korlátok azonban idővel változhatnak.

## <a name="partitioning-azure-table-storage"></a>Az Azure Table Storage particionálása
Az Azure Table Storage egy particionálási célokra kifejlesztett kulcs-érték tároló. Minden entitás tárolása egy partícióban történik, és a partíciókat belsőleg az Azure Table Storage kezeli. A táblákban tárolt minden entitásnak meg kell adnia egy kétrészes kulcsot, amely a következőkből épül fel:

* **A partíciókulcs**. Ez egy sztringérték, amely azt adja meg, hogy az Azure Table Storage melyik partícióba helyezi az entitást. Az azonos partíciókulccsal rendelkező entitások mind ugyanazon a partíción találhatók.
* **A sorkulcs**. Ez egy másik sztringérték, amely az entitást azonosítja a partíción belül. A partícióban lévő entitások a sorkulcs alapján vannak növekvő betűrendbe rendezve. Minden egyes entitásnak egyedi partíciókulcs-sorkulcs kombinációval kell rendelkeznie, amelynek a hossza legfeljebb 1 KB lehet.

Az entitásokhoz tartozó többi adat az alkalmazás által meghatározott mezőket tartalmazza. Nincs semmilyen séma kikényszerítve, és minden egyes sor más és más alkalmazás-specifikus mezők halmazát tartalmazhatja. Az egyetlen megkötés, hogy az egyes entitások maximális mérete (beleértve a partíció- és sorkulcsot is) jelenleg legfeljebb 1 MB lehet. A tábla maximális mérete 200 TB, bár ezek az értékek a későbbiekben változhatnak. (A korlátokkal kapcsolatos legfrissebb információkat [Az Azure Storage skálázhatósági és teljesítménycéljai] bemutató oldalon ellenőrizheti a Microsoft webhelyén.)

Amennyiben ezt meghaladó méretű entitásokat próbál letárolni, esetleg több táblára felosztva teheti ezt meg. Vertikális particionálás használatával ossza fel a mezőket csoportokra aszerint, hogy várhatóan melyik mezőkhöz férnek hozzá együtt.

A 7. ábra egy fiktív e-kereskedelmi alkalmazás példatárfiókjának (Contoso Data) logikai szerkezetét mutatja be. A tárfiók három táblát tartalmaz: Customer Info (ügyféladatok), Product Info (termékadatok) és Order Info (megrendelésadatok). Mindegyik tábla több partíciót tartalmaz.

A Customer Info táblában az adatok a felhasználó városa szerint vannak particionálva, a sorkulcs pedig az ügyfél-azonosító. A Product Info táblában a termékek a termékkategória szerint vannak particionálva, a sorkulcs pedig a termékszám. Az Order Info táblában a megrendelések a feladásuk dátuma szerint vannak particionálva, a sorkulcs pedig a megrendelések fogadásának időpontja. Az adatok az egyes partíciókban a sorkulcs alapján vannak rendezve.

![Táblák és partíciók egy példatárfiókban](./images/data-partitioning/TableStorage.png)

*7. ábra Táblák és partíciók egy példatárfiókban*

> [!NOTE]
> Az Azure Table Storage minden entitáshoz felvesz egy időbélyegmezőt is. Az időbélyegmezőt a Table Storage tartja karban, és minden alkalommal frissíti, amikor az entitást módosítják és visszaírják valamelyik partícióba. A Table Storage szolgáltatás ennek a mezőnek a használatával valósítja meg az optimista egyidejűséget. (Minden alkalommal, amikor az alkalmazás visszaírja valamelyik entitást a táblatárolóba, a Table Storage szolgáltatás összehasonlítja az entitás időbélyegzőjének értékét a táblatárolóban tárolt értékkel. Ha az értékek különböznek, ez azt jelenti, hogy egy másik alkalmazás módosította az entitást annak utolsó lekérése óta, és az írási művelet meghiúsul. Ne módosítsa ezt a mezőt az alkalmazás saját kódjában, és ne is adja meg az értékét az új entitások létrehozásakor.
>
>

Az Azure Table Storage a partíciókulcs alapján határozza meg az adatok tárolásának módját. Ha egy korábban nem használt partíciókulccsal rendelkező entitást vesz fel egy táblába, az Azure Table Storage egy új partíciót hoz létre az entitás számára. Az ezzel a partíciókulccsal rendelkező többi entitás is ugyanezen a partíción lesz tárolva.

Ez az eljárás lényegében egy automatikus horizontális felskálázási stratégiát valósít meg. Mindegyik partíció egyetlen kiszolgálón helyezkedik el az Azure-adatközpontban annak biztosításához, hogy az adatokat egyetlen partícióról lekérő lekérdezések gyorsan futhassanak. A különböző partíciók azonban esetleg több kiszolgálóra annak leosztva. Emellett egyetlen kiszolgáló több partíciót is futtathat, ha a partíciók mérete korlátozva van.

Az Azure Table Storage-ben használt entitások kialakításakor vegye figyelembe a következő szempontokat:

* A partíciókulcs és a sorkulcs értékeit az adatelérés módja alapján kell meghatározni. Válasszon olyan partíciókulcs-sorkulcs kombinációt, amely a lekérdezések nagyobb részét támogatja. A leghatékonyabb lekérdezések a partíciókulcs és a sorkulcs megadásával kérik le az adatokat. Az egy partíciókulcsot és a sorkulcsok egy tartományát megadó lekérdezések egy partíció vizsgálatával hajthatók végre. Ez viszonylag gyorsan megy, mivel az adatok a sorkulcs szerint vannak rendezve. Ha a lekérdezésben nincs megadva, hogy melyik partíciót kell átvizsgálni, a partíciókulcs alapján az Azure Table Storage-nek esetleg minden partíciót át kell vizsgálnia az adatok után.

  > [!TIP]
  > Ha valamely entitás természetes kulccsal rendelkezik, használja azt partíciókulcsként, és adjon meg egy üres sztringet sorkulcsként. Ha egy entitás két tulajdonságból álló összetett kulccsal rendelkezik, használja a ritkábban változó tulajdonságot partíciókulcsként, a másikat pedig sorkulcsként. Ha egy entitás kettőnél több kulcstulajdonsággal rendelkezik, a tulajdonságok összefűzésével adja meg a partíció- és a sorkulcsot.
  >
  >
* Ha rendszeresen hajt végre olyan lekérdezéseket, amelyek a partíció- és a sorkulcstól eltérő mezők alapján keresik az adatokat, vegye fontolóra az [Indextábla minta] használatát.
* Ha a partíciókulcsokat egy monoton növekvő vagy csökkenő sorozat alapján (például 0001, 0002, 0003 és így tovább) hozza létre, és minden partíció csak korlátozott mennyiségű adatot tartalmaz, az Azure Table Storage képes fizikailag csoportosítani ezeket a partíciókat egyazon kiszolgálóra. Ez a mechanizmus feltételezi, hogy az alkalmazás valószínűleg partíciók egymást követő összefüggő tartományán hajtja majd végre a lekérdezéseket (tartománylekérdezések), és erre a forgatókönyvre van optimalizálva. Ez a megközelítés azonban kritikus pontokat eredményezhet egyes kiszolgálók körül, mivel az új entitások beszúrása valószínűleg az összefüggő tartomány egyik vagy másik végén koncentrálódik majd. Emellett a skálázhatóságot is csökkentheti. A terhelés kiszolgálók közötti egyenletesebb elosztása érdekében érdemes kivonatolnia a partíciókulcsot, hogy a sorozat véletlenszerűbb legyen.
* Az Azure Table Storage támogatja a tranzakciós műveletek végrehajtását az egyazon partíción lévő entitásokon. Ez azt jelenti, hogy az alkalmazás egyidejűleg több beszúrás, frissítés, törlés, csere vagy egyesítés műveletet hajthat végre egyetlen elemi egységként (feltéve, hogy a tranzakció nem tartalmaz 100-nál több entitást és a kérelem hasznos adattartalma nem haladja meg a 4 MB méretet). A több partíciót érintő műveletek nem tranzakciószintűek, és ezért lehetséges, hogy végső konzisztenciát kell megvalósítania az [Adatkonzisztencia – Ismertető] szakaszban leírtak szerint. A Table Storage és a tranzakciók részletesebb leírását a Microsoft webhelyén, [Entitáscsoport-tranzakciók végrehajtása] foglalkozó oldalon találja.
* Gondosan mérlegelje a partíciókulcs részletességét a következő okok miatt:
  * Ha minden entitás ugyanazzal a partíciókulccsal rendelkezik, a Table Storage szolgáltatás egyetlen nagy partíciót hoz létre egyetlen kiszolgálón. Ez megakadályozza a horizontális felskálázást, és a terhelés egy kiszolgálóra összpontosul. Ez a megközelítés ezért csak olyan rendszerekhez alkalmas, amelyek kis számú entitást kezelnek. Mindazonáltal ez a megközelítés biztosítja, hogy valamennyi entitás részt vehet entitáscsoport-tranzakciókban.
  * Ha minden entitás egyedi partíciókulccsal rendelkezik, a Table Storage szolgáltatás egy külön partíciót hoz létre mindegyik entitás számára, és így nagy számú kis partíció keletkezhet (az entitások méretétől függően). Ez a megközelítés jobban skálázható, mint az, amelyik egyetlen partíciókulcsot használ, de az entitáscsoportokon végzett tranzakciók nem lehetségesek. Emellett a több entitást érintő lekérdezések esetén esetleg több kiszolgálót is olvasni kell. Azonban ha az alkalmazás tartománylekérdezéseket végez, a lekérdezések optimalizálhatóak, ha a partíciókulcsokat monoton sorozat mentén hozza létre.
  * Ha az entitások egy részhalmazához ugyanazt a partíciókulcsot használja, a kapcsolódó entitások egy partícióba csoportosíthatóak. A kapcsolódó entitásokat érintő műveletek végrehajthatók entitáscsoport-tranzakciók használatával, és a több kapcsolódó entitást olvasó lekérdezések egyetlen kiszolgáló elérésével teljesíthetőek.

Az adatok Azure Table Storage-ben való particionálásra vonatkozó további információkat [Az Azure Storage Table tervezési útmutatója] találja a Microsoft webhelyén.

## <a name="partitioning-azure-blob-storage"></a>Az Azure Blob Storage particionálása
Az Azure Blob Storage segítségével nagyméretű bináris objektumok tárolhatóak – jelenleg akár 5 TB méretű blokkblobok vagy 1 TB méretű lapblobok. (A legfrissebb információkat [Az Azure Storage skálázhatósági és teljesítménycéljai] bemutató oldalon ellenőrizheti a Microsoft webhelyén.) A blokkblobok olyan forgatókönyvekben használhatóak, ahol nagy mennyiségű adatot kell gyorsan fel- vagy letölteni. A lapblobok olyan alkalmazásokban használhatóak, ahol nem sorban, hanem véletlenszerűen kell elérni az adathalmaz egyes részeit.

Minden blob (blokk vagy lap) külön tárolóban található egy Azure-tárfiókban. A tárolók használatával csoportokba rendezhetőek a hasonló biztonsági követelményekkel rendelkező kapcsolódó blobok. A csoportosítás nem fizikai, hanem logikai alapon történik. A tárolókon belül minden egyes blob egyedi névvel rendelkezik.

A blobok partíciókulcsa a fiók neve + a tároló neve + a blob neve. Ez azt jelenti, hogy minden egyes blob saját partícióval rendelkezhet, ha a blob terhelése megköveteli. A blobok leoszthatóak több kiszolgálóra a hozzáférés horizontális felskálázása érdekében, egy blobot azonban csak egyetlen kiszolgáló szolgálhat ki. 

Az egyetlen blokk (blokkblob) vagy lap (lapblob) írására vonatkozó műveletek elemiek, a több blokkra, lapra vagy blobra vonatkozóak azonban nem elemiek. Ha biztosítania kell a blokkokon, lapokon és blobokon végzett írási műveletek során a konzisztenciát, blobbérlet alkalmazásával zárolhatja az írásukat.

Az Azure Blob Storage átviteli sebességre vonatkozó célértéke blobonként 60 MB vagy 500 kérelem másodpercenként. Ha várhatóan túllépi ezeket a korlátokat, és a blobban lévő adatok viszonylag statikusak, érdemes lehet az Azure Content Delivery Network használatával replikálni a blobokat. További információkat az [Azure Content Delivery Network] oldalon talál a Microsoft webhelyén. További útmutatást és szempontokat [Az Azure Content Delivery Network használata] ismertető cikkben talál.

## <a name="partitioning-azure-storage-queues"></a>Az Azure tároló-üzenetsorok particionálása
Az Azure tároló-üzenetsorok használatával folyamatok közötti aszinkron üzenetkezelés valósítható meg. Minden egyes Azure-tárfiók korlátlan számú üzenetsort, és mindegyik üzenetsor korlátlan számú üzenetet tartalmazhat. Az egyetlen korlát a tárfiókban rendelkezésre álló hely mennyisége. Egy egyes üzenetek maximális mérete 64 KB. Ha ennél nagyobb méretű üzenetekre van szüksége, vegye fontolóra az Azure Service Bus-üzenetsorok használatát.

Minden egyes üzenetsor egyedi névvel rendelkezik a tárfiókon belül, amelyben található. Az Azure név alapján particionálja az üzenetsorokat. Az egy üzenetsorba tartozó üzeneteket egy partíció tárolja, amelyet egy kiszolgáló vezérel. A különböző üzenetsorok a terhelés elosztása érdekében futtathatók külön kiszolgálókon. Az üzenetsorok kiszolgálókra való leosztása az alkalmazások és a felhasználók számára transzparens módon történik.

 A nagy alkalmazásokban ne ugyanazt az üzenetsort használja az alkalmazás összes példányához, mivel így az üzenetsort futtató kiszolgáló kritikus ponttá válhat. Használjon inkább külön üzenetsorokat az alkalmazás különböző funkcionális területei szerint. Az Azure tároló-üzenetsorok nem támogatják a tranzakciókat, ezért ha az üzeneteket külön üzenetsorokra irányítja, ez nem befolyásolja az üzenetkonzisztenciát.

Az egyes Azure tároló-üzenetsorok másodpercenként 2000 üzenetet képesek feldolgozni.  Ha ennél nagyobb sebességre van szüksége, érdemes több üzenetsort létrehoznia. Például egy globális alkalmazás esetében hozzon létre külön tároló-üzenetsorokat külön tárfiókokban az egyes régiókban futó alkalmazáspéldányok kezelésére.

## <a name="partitioning-strategies-for-azure-service-bus"></a>Az Azure Service Bus particionálási stratégiái
Az Azure Service Bus egy üzenetközvetítő használatával dolgozza fel a Service Bus-üzenetsorokra vagy -témakörökbe küldött üzeneteket. Alapértelmezés szerint az üzenetsorra vagy témakörbe küldött összes üzenetet ugyanaz az üzenetközvetítő folyamat kezeli. Az ilyen architektúra korlátozhatja az üzenetsor teljes feldolgozási sebességét. Az üzenetsorok és témakörök azonban a létrehozásukkor particionálhatóak. Ezt az üzenetsor vagy témakör *EnablePartitioning* tulajdonságának *true* értékre állításával teheti meg.

A particionált üzenetsor vagy témakör több töredékre osztható fel, amelyek mindegyikét egy külön üzenettár és üzenetközvetítő szolgálja ki. A töredékek létrehozása és felügyelete a Service Bus feladata. Amikor egy alkalmazás üzenetet küld egy particionált üzenetsorra vagy témakörbe, a Service Bus hozzárendeli az üzenetet az üzenetsor vagy témakör valamelyik töredékéhez. Amikor egy alkalmazás üzenetet fogad egy üzenetsorról vagy előfizetésről, a Service Bus az összes töredékben ellenőrzi, hogy melyik a következő elérhető üzenet, és azt továbbítja az alkalmazásba feldolgozásra.

Ez a struktúra segít elosztani a terhelést az üzenetközvetítők és üzenettárak közt, így növeli a skálázhatóságot és javítja a rendelkezésre állást. Ha valamelyik töredék üzenetközvetítője vagy üzenettára átmenetileg nem érhető el, a Service Bus a többi elérhető töredék valamelyikéből ki tudja olvasni az üzeneteket.

A Service Bus az üzeneteket az alábbiak szerint osztja le a töredékekre:

* Ha az üzenet egy munkamenet részét képezi, az azonos értékű *SessionId* tulajdonsággal rendelkező üzenetek ugyanarra a töredékre lesznek küldve.
* Ha az üzenet nem képezi munkamenet részét, de a feladó megadta a *PartitionKey* tulajdonság értéket, az összes azonos *PartitionKey* értékű üzenet ugyanarra a töredékre lesz küldve.

  > [!NOTE]
  > Ha a *SessionId* és a *PartitionKey* tulajdonságok értéke egyaránt meg van adva, ezeket ugyanarra az értékre kell állítani, vagy az üzenet el lesz utasítva.
  >
  >
* Ha egy üzenet *SessionId* és *PartitionKey* tulajdonságai nincsenek megadva, de a duplikálásészlelés engedélyezve van, a rendszer a *MessageId* tulajdonságot használja. Az azonos *MessageId* értékű üzenetek ugyanarra a töredékre lesznek küldve.
* Ha az üzeneteknek nincs *SessionId, PartitionKey* vagy *MessageId* tulajdonsága, a Service Bus szekvenciálisan osztja le az üzeneteket a töredékekre. Ha egy töredék nem érhető el, a Service Bus továbblép a következőre. Ez azt jelenti, hogy az üzenetkezelési infrastruktúra ideiglenes hibája nem okozza az üzenetküldési művelet meghiúsulását.

A Service Bus-üzenetsorok vagy -témakörök particionálásának tervezése során mérlegelje a következőket:

* A Service Bus-üzenetsorok és -témakörök egy Service Bus-névtér hatókörén belül jönnek létre. A Service Bus névterenként jelenleg 100 particionált üzenetsort vagy témakört képes kezelni.
* Az egyes Service Bus-névterek kvótákat írnak elő a rendelkezésre álló erőforrásokra, például az egyes témakörök előfizetőinek számára, az egyidejű küldési és fogadási kérelmek számára, valamint a létrehozható egyidejű kapcsolatok maximális számára vonatkozóan. A kvóták dokumentált értékeit a Microsoft webhelyén, a [Service Bus-kvóták] ismertető lapon találja. Ha várhatóan túllépi majd ezeket az értékeket, hozzon létre további névtereket a maguk üzenetsoraival és témaköreivel, és ossza le a terhelést ezekre a névterekre. Például egy globális alkalmazás esetében hozzon létre külön névtereket mindegyik régióban, és konfigurálja az alkalmazáspéldányokat, hogy a legközelebbi névtér üzenetsorait és témaköreit használják.
* A tranzakciók keretében küldött üzenetekben meg kell adni egy partíciókulcsot. Ez lehet egy *SessionId*, egy *PartitionKey* vagy egy *MessageId* tulajdonság. Az egy tranzakció keretében küldött üzenetekben ugyanazt a partíciókulcsot kell megadni, mivel ugyanannak az üzenetközvetítő folyamatnak kell feldolgoznia azokat. Az egy tranzakcióba tartozó üzeneteket nem küldheti külön üzenetsorokra vagy témakörökbe.
* A particionált üzenetsorok és témakörök nem konfigurálhatóak automatikus törlésre tétlenség esetére.
* A particionált üzenetsorok és témakörök jelenleg nem használhatóak az Advanced Message Queueing Protocol (AMQP) protokollal, ha platformfüggetlen vagy hibrid megoldásokat épít fel.

## <a name="partitioning-strategies-for-cosmos-db"></a>A Cosmos DB particionálási stratégiái

Az Azure Cosmos DB egy NoSQL-adatbázis, amely az [Azure Cosmos DB SQL API][cosmosdb-sql-api] használatával képes JSON-dokumentumokat tárolni. A Cosmos DB-adatbázisban lévő objektumok vagy egyéb adatok JSON-szerializált ábrázolásai. A rendszer nem tartat be rögzített sémákat, ez alól az egyetlen kivétel az, hogy minden dokumentumnak tartalmaznia kell egy egyedi azonosítót.

A dokumentumok gyűjteményekbe vannak rendezve. A kapcsolódó dokumentumokat gyűjteménybe foglalhatja. Például egy blogbejegyzéseket karbantartó rendszerben mindegyik blogbejegyzés tartalmát külön dokumentumban tárolhatja egy gyűjteményben. Az egyes tématípusok szerint is létrehozhat gyűjteményeket. Egy több-bérlős alkalmazásban, például egy olyan rendszerben, ahol több szerző is kezelheti és irányíthatja a saját blogbejegyzéseit, a blogokat particionálhatja szerző szerint, és mindegyik szerző számára külön gyűjteményeket hozhat létre. A gyűjtemények számára lefoglalt tárhely rugalmas, szükség szerint csökkenthető vagy növelhető.

A Cosmos DB támogatja az adatok automatikus particionálását az alkalmazásban definiált partíciókulcs alapján. A *logikai partíció* egy olyan partíció, amely egy adott partíciókulcs-értékhez tartozó adatokat tárolja. Az adott partíciókulcs-értékkel rendelkező összes dokumentum ugyanabba a logikai partícióba kerül. A Cosmos DB az értékeket a partíciókulcs kivonata alapján osztja el. Az egyes logikai partíciók maximális mérete 10 GB. Így a partíciókulcs megfelelő megválasztása fontos szempont a tervezés során. Válasszon egy olyan tulajdonságot, amelynek az értékei és a hozzáférési mintái széles skálán mozognak. További információkat az [Azure Cosmos DB particionálási és horizontális leskálázási eljárásait](/azure/cosmos-db/partition-data) ismertető cikkben talál.

> [!NOTE]
> Mindegyik Cosmos DB-adatbázis rendelkezik *teljesítményszinttel*, amely az adatbázis számára elérhető erőforrások mennyiségét határozza meg. A teljesítményszinthez egy *kérelemegység* (RU) sebességkorlát tartozik. Az RU-sebességkorlát az adott gyűjtemény számára fenntartott, kizárólag általa használható erőforrások mennyiségét határozza meg. A gyűjtemény költségei az adott gyűjtemény számára kiválasztott teljesítményszinttől függenek. A nagyobb teljesítményszint (és RU-sebességkorlát) nagyobb költségekkel jár. A gyűjtemények teljesítményszintjét az Azure Portalon módosíthatja. További információkat [az Azure Cosmos DB kérelemegységeit ismertető][cosmos-db-ru] cikkben talál.
>
>

Ha a Cosmos DB által biztosított particionálás mechanizmus nem bizonyul elegendőnek, az adatokat esetleg az alkalmazás szintjén kell szegmensekre osztania. A dokumentumgyűjtemények egy természetes mechanizmust kínálnak az adatok particionálására egy adott adatbázisban. A szegmensekre particionálás legegyszerűbben úgy valósítható meg, ha mindegyik szegmenshez létrehoz egy gyűjteményt. A tárolók logikai erőforrások, és több kiszolgálóra is kiterjedhetnek. A rögzített méretű tárolók mérete legfeljebb 10 GB, feldolgozási sebessége legfeljebb 10000 RU/s lehet. A nem korlátozott tárolók nem rendelkeznek tárhelykorláttal, azonban meg kell adni hozzájuk egy partíciókulcsot. Az alkalmazások horizontális particionálása esetén az ügyfélalkalmazásnak a kérelmeket a megfelelő szegmensre kell irányítania, ez általában az adatok a szegmenskulcsot meghatározó egyes attribútumain alapuló leképezési mechanizmussal valósítható meg. 

Az összes adatbázis egy Cosmos DB-adatbázis adatbázisfiók környezetében jön létre. Egy fiók több adatbázist is tartalmazhat, és meghatározza, hogy az adatbázisok mely régiókban lesznek létrehozva. Mindegyik fiók betartatja a saját hozzáférés-vezérlését. A Cosmos DB fiókok használatával a szegmenseket (az adatbázisokon belüli gyűjteményeket) az azokat használó felhasználókhoz földrajzilag közel helyezheti el, és kikényszerítheti, hogy csak az adott felhasználók érhessék el azokat.

Ha az adatokat a Cosmos DB SQL API-val tervezi particionálni, vegye figyelembe a következő szempontokat:

* **A Cosmos DB-adatbázis számára elérhető erőforrások mennyisége a fiók kvótakorlátainak függvénye**. Mindegyik adatbázis több gyűjteményt is tárolhat, és mindegyik gyűjteményhez tartozik egy teljesítményszint, amely az adott gyűjtemény RU-sebességkorlátját (a fenntartott feldolgozási sebességet) határozza meg. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][azure-limits].
* **Mindegyik dokumentumnak rendelkeznie kell egy attribútummal, amely alapján egyedileg azonosítható a gyűjteményen belül**. Ez az attribútum nem azonos a partíciókulccsal, amely azt határozza meg, hogy melyik gyűjteményben található a dokumentum. Egy gyűjtemény nagy számú dokumentumot tartalmazhat. Ezt elméletileg csak a dokumentumazonosító hossza korlátozza. A dokumentumazonosító legfeljebb 255 karakter hosszúságú lehet.
* **A dokumentumokra irányuló műveletek egy tranzakció környezetében lesznek végrehajtva. A tranzakciók hatóköre a dokumentumot tároló gyűjtemény.** Ha egy művelet meghiúsul, az általa végrehajtott feldolgozás vissza lesz vonva. Az épp feldolgozás alatt álló dokumentumokon végrehajtott módosítások pillanatképszinten el vannak különítve. Ez a mechanizmus biztosítja, hogy ha például egy új dokumentum létrehozására irányuló kérelem meghiúsul, az adatbázist egyidejűleg lekérdező többi felhasználó nem látja a félkész, majd eltávolított dokumentumot.
* **Az adatbázis-lekérdezések hatóköre szintén a gyűjtemény**. Egy lekérdezés csak egyetlen gyűjtemény adatait kérdezheti le. Ha több gyűjteményből kell lekérdeznie adatokat, mindegyik gyűjteményt külön-külön kell lekérdeznie, majd az eredményeket az alkalmazás kódjában egyesítenie.
* **A Cosmos DB támogatja a programozható elemek használatát, amelyek a gyűjteményben a dokumentumok mellett tárolhatóak**. Ezek lehetnek tárolt eljárások, felhasználó által definiált függvények és eseményindítók (JavaScript nyelven). Ezek az elemek a gyűjteményen belül az összes dokumentumot elérhetik. Ezen felül ezek az elemek a környezeti tranzakció hatókörében (egy, valamely dokumentumon végrehajtott létrehozás, törlés vagy csere művelet eredményeként aktiválódó trigger esetén) vagy egy új tranzakció indításával (egy, valamely kifejezett ügyfélkérelem eredményeképp futtatott tárolt eljárás esetén) futtatható. Ha egy programozható elem kódja kivételt dob, a tranzakció vissza lesz vonva. A tárolt eljárások és eseményindítók használatával tartható fent az integritás és a konzisztencia a dokumentumok közt, de a dokumentumoknak ugyanabba a gyűjteménybe kell tartoznia.
* **Törekedni kell rá, hogy az adatbázisban tárolni kívánt gyűjtemények ne haladják meg a teljesítményszintjeik által meghatározott feldolgozási sebességeket**. További információkat [az Azure Cosmos DB kérelemegységeit ismertető][cosmos-db-ru] cikkben talál. Ha várhatóan meghaladja majd ezeket a korlátokat, érdemes a gyűjteményeket különböző fiókokban lévő adatbázisokra szétosztani a terhelés csökkentése érdekében.

## <a name="partitioning-strategies-for-azure-search"></a>Az Azure Search particionálási stratégiái
Az adatkeresési funkcionalitás a webalkalmazások biztosított leggyakoribb navigációs és felfedezési módszer. Segítségével a felhasználók gyorsan találhatják meg az erőforrásokat (például a termékeket egy e-kereskedelmi alkalmazásban) a keresési feltételek kombinációja alapján. Az Azure Search szolgáltatás teljes szöveges keresési funkcionalitást biztosít a webes tartalmakban, és olyan szolgáltatásokat kínál, mint a szövegkiegészítés, a közeli találatokon alapuló lekérdezési javaslatok és a jellemzőalapú navigáció. Ezeknek a képességeknek a teljes leírását a [Mi az az Azure Search?] című oldalon találja a Microsoft webhelyén.

Az Azure Search a kereshető tartalmakat JSON-dokumentumokként tárolja egy adatbázisban. Definiálhat indexeket a dokumentumok kereshető mezőinek a meghatározásához, majd ezeket a definíciókat megadhatja az Azure Searchnek. Amikor egy felhasználó beküld egy keresési kérelmet, az Azure Search a megfelelő indexek használatával találja meg az egyező elemeket.

A versengés csökkentése érdekében az Azure Search által használt tárterület 1, 2, 3, 4, 6 vagy 12 partícióra osztható, és mindegyik partíció 6 példányban replikálható. A partíciók és a replikák számának szorzatát *keresési egységnek* (SU) nevezzük. Az Azure Search egyetlen példánya legfeljebb 36 SU-t tartalmazhat (tehát egy 12 partícióval rendelkező adatbázis legfeljebb 3 replikát támogat).

A számlázás a szolgáltatásban lefoglalt SU-k száma alapján történik. A kereshető tartalom mennyiségének vagy a keresési kérelmek számának növekedtével a meglévő Azure Search-példányok SU-inak száma is növelhető a többletterhelés kezelése érdekében. Maga az Azure Search a dokumentumokat egyenletesen osztja el a partíciók közt. A manuális particionálási stratégiák jelenleg nem támogatottak.

Mindegyik partíció legfeljebb 15 millió dokumentumot tartalmazhat vagy 300 GB tárterületet foglalhat el (amelyik érték kisebb). Legfeljebb 50 indexet hozhat létre. A szolgáltatás teljesítménye a dokumentumok összetettségétől, a rendelkezésre álló indexektől és a hálózati késés hatásaitól függően változik. Átlagosan egyetlen replika (1 SU) másodpercenként 15 lekérdezést (QPS) képes feldolgozni, bár javasolt saját adatok használatával teljesítménymérést végezni a feldolgozási sebesség pontosabb mérése érdekében. További információkért lásd [Az Azure Search szolgáltatási korlátozásai] foglalkozó oldalt a Microsoft webhelyén.

> [!NOTE]
> A kereshető dokumentumokban tárolható adatok típusa egy korlátozott halmazba tartozhat: sztringek, logikai értékek, numerikus adatok, dátum és idő adatok és egyes földrajzi adatok. További információkért lásd [Támogatott adattípusok (Azure Search)] foglalkozó oldalt a Microsoft webhelyén.
>
>

Csak korlátozottan szabályozható, hogy az Azure Search hogyan particionálja az adatokat a szolgáltatás egyes példányaiban. Azonban egy globális környezetben esetleg növelhető a teljesítmény, valamint csökkenthető a késés és a versengés magának a szolgáltatásnak a particionálásával a következő stratégiák valamelyike mentén:

* Hozzon létre egy Azure Search-példányt minden egyes földrajzi régióban, és gondoskodjon róla, hogy az ügyfélalkalmazások a legközelebbi elérhető példányra legyenek irányítva. Ehhez a stratégiához a kereshető tartalmak frissítéseit időben kell replikálni a szolgáltatás minden példányán.
* Hozzon létre két szintet az Azure Searchben:

  * Egy helyi szolgáltatást minden egyes régióban, amely az adott régióban lévő felhasználók által leggyakrabban lekérdezett adatokat tartalmazza. A felhasználók a kérelmeket ide irányítva gyors, de korlátozott eredményeket kaphatnak.
  * Egy globális szolgáltatást, amely az összes adatot tartalmazza. A felhasználók a kérelmeket ide irányítva lassabb, de teljesebb eredményeket kaphatnak.

Ez a megközelítés akkor a legmegfelelőbb, ha a keresett adatok az egyes régiókban jelentősen eltérnek.

## <a name="partitioning-strategies-for-azure-redis-cache"></a>Az Azure Redis Cache particionálási stratégiái
Az Azure Redis Cache egy, a Redis kulcs-érték adattáron alapuló megosztott gyorsítótárazási szolgáltatást biztosít a felhőben. Ahogy a neve is mutatja (cache = gyorsítótár), az Azure Redis Cache egy gyorsítótárazási megoldásnak készült. Csak átmeneti adatok tárolására készült, és nem állandó adattárként szolgál. Az Azure Redis Cache-t használó alkalmazásokat úgy érdemes kialakítani, hogy tovább működjenek, ha a gyorsítótár nem érhető el. Az Azure Redis Cache támogatja az elsődleges/másodlagos replikációt a magas rendelkezésre állás biztosítása érdekében, de jelenleg a gyorsítótár maximális mérete 53 GB-ra van korlátozva. Ha ennél több tárhely szükséges, további gyorsítótárakat kell létrehoznia. További információkat az [Azure Redis Cache] oldal tartalmaz a Microsoft webhelyén.

A Redis-adattárak particionálása szétosztja az adatokat a Redis szolgáltatás példányai között. Minden példány egy külön partíciót alkot. Az Azure Redis Cache a Redis-szolgáltatásokat egy előtár mögött kivonatolja, és nem teszi azokat közvetlenül közzé. A particionálás legegyszerűbben úgy valósítható meg, ha létrehoz több Azure Redis Cache-példányt, és az adatokat azok között osztja el.

Ez egyes adatelemekhez hozzárendelhet egy azonosítót (a partíciókulcsot), amely meghatározza, hogy melyik gyorsítótár tárolja az adatelemet. Az ügyfélalkalmazás logikája azután ennek az azonosítónak a használatával irányíthatja a kérelmeket a megfelelő partícióra. Ez a séma nagyon egyszerű, de ha a particionálási séma módosul (például további Azure Redis Cache-példányok létrehozása esetén), az ügyfélalkalmazások újrakonfigurálására lehet szükség.

A natív Redis (nem az Azure Redis Cache) támogatja a Redis-fürtözésen alapuló kiszolgálóoldali particionálást. Ebben a megközelítésben az adatok egyenletesen oszthatók meg a kiszolgálók között egy kivonatoló mechanizmus segítségével. Mindegyik Redis-kiszolgáló tárolja a partíción tárolt kivonatkulcsokat leíró metaadatokat, és arra vonatkozóan is tartalmaz információkat, hogy mely kivonatkulcsok találhatók a más kiszolgálókon lévő partíciókon.

Az ügyfélalkalmazások egyszerűen beküldik a kérelmeket valamelyik résztvevő Redis-kiszolgálóra (valószínűleg a legközelebbire). A Redis-kiszolgáló megvizsgálja az ügyfélkérelmet. Ha az helyileg megoldható, akkor elvégzi a kért műveletet. Ha nem oldható meg, továbbítja a kérést a megfelelő kiszolgálóra.

Erről a Redis-fürtözés használatával megvalósított modellről további részleteket a Redis webhelyén, a [Oktatóanyag Redis-fürtökhöz] tartalmazó oldalon talál. A Redis-fürtözés az ügyfélalkalmazások számára transzparens módon történik. A fürthöz további Redis-kiszolgálók is hozzáadhatók (és az adatok újraparticionálhatók) anélkül, hogy az ügyfeleket újra kellene konfigurálni.

> [!IMPORTANT]
> Az Azure Redis Cache jelenleg nem támogatja a Redis-fürtözést. Ha szeretné megvalósítani ezt a megközelítést az Azure-ban is, saját Redis-kiszolgálókat kell kialakítania a Redis Azure-beli virtuális gépekre való telepítésével és manuális konfigurálásával. A Microsoft webhelyének [A Redis futtatása CentOS Linux rendszerű virtuális gépen az Azure-ban] tárgyaló oldala egy példán keresztül mutatja be, hogyan hozható létre és konfigurálható egy Azure-beli virtuális gépként futó Redis-csomópont.
>
>

A Redis használatával történő particionálásról további információkat a Redis webhelyén elérhető, [Particionálás: adatok felosztása több Redis-példány között] leíró oldalon talál. A szakasz további része feltételezi, hogy ügyféloldali vagy proxyval támogatott particionálást végez.

Ha az adatokat az Azure Redis Cache-sel tervezi particionálni, vegye figyelembe a következő szempontokat:

* Az Azure Redis Cache nem végleges adattárolásra lett kialakítva, ezért bármilyen particionálási sémát választ is, az alkalmazás kódjának az adatokat egy, a gyorsítótártól különböző helyről kell tudnia lekérdezni.
* A gyakran együtt használt adatokat érdemes egy partíción tartani. A Redis egy hatékony kulcs-érték tároló, amely több magas szinten optimalizált mechanizmust biztosít az adatok rendszerezéséhez. Ezek a mechanizmusok a következők lehetnek:

  * Egyszerű sztringek (legfeljebb 512 MB hosszúságú bináris adatok)
  * Összesített típusok, például listák (amelyek szolgálhatnak üzenetsorként vagy veremként)
  * Halmazok (rendezett és rendezetlen)
  * Kivonatok (amelyekkel csoportosíthatóak a kapcsolódó mezők, például az egy objektum mezőit jelölő elemek)
* Az összesített típusok összerendelhető segítségével több kapcsolódó mező ugyanazzal a kulccsal. A Redis-kulcsok listákat, halmazokat vagy kivonatokat azonosítanak, nem pedig az azokban tárolt adatelemeket. Ezek a típusok az Azure Redis Cache-ben mind elérhetők, és a Redis webhelyén, az [Adattípusok] lapon ismerhetőek meg. Például egy e-kereskedelmi rendszer az ügyfelek által feladott megrendeléseket követő részében az egyes ügyfelek adatai tárolhatóak egy Redis-kivonatban, amelynek a kulcsa lehet a felhasználó azonosítója. Mindegyik kivonat az ügyfél megrendelésazonosítóinak egy gyűjteményét tárolja. Egy külön Redis-halmaz tárolhatja a megrendeléseket, ezeket is kivonatként strukturálva, és kulcsként a megrendelésazonosítót használva. A 8. ábrán ez a struktúra látható. Vegye figyelembe, hogy a Redis nem valósít meg semmilyen hivatkozásintegritást, így a fejlesztő feladata, hogy fenntartsa az ügyfelek és a megrendelések kapcsolatát.

![Javasolt struktúra a Redis-tárolóban a megrendelések és adataik rögzítéséhez](./images/data-partitioning/RedisCustomersandOrders.png)

*8. ábra Javasolt struktúra a Redis-tárolóban a megrendelések és adataik rögzítéséhez*

> [!NOTE]
> A Redisben a kulcsok bináris adatértékek (mint a Redis-sztringek), és legfeljebb 512 MB adatot tartalmazhatnak. Elméletileg a kulcsok szinte bármilyen információt tartalmazhatnak. Javasoljuk azonban egy egységes kulcselnevezési konvenció bevezetését, amely leírja az adatok típusát és azonosítja az entitásokat, de nem túl hosszú. Általános megközelítésként javasolt „entitástípus:azonosító” formátumú kulcsokat használni. Például a „user:99” névvel jelölheti egy 99-es azonosítójú ügyfél kulcsát.
>
>

* A vertikális particionálás megvalósítható, ha a különböző összesítésekben lévő kapcsolódó információkat ugyanabban az adatbázisban tárolja. Például egy e-kereskedelmi alkalmazásban tárolhatja a termékekkel kapcsolatos gyakran használt adatokat az egyik Redis-kivonatban, a kevésbé gyakran használt felhasználóadatokat pedig egy másikban.
  Mindkét kivonat használhatja ugyanazt a termékazonosítót a kulcs részeként. Használhatja például "termék: *Neurális hálózat*" (ahol *Neurális hálózat* a termékazonosító) a termékinformációkhoz és a "product_details: *Neurális hálózat*" a részletes adatokhoz. Ezzel a stratégiával csökkenthető a legtöbb lekérdezés által várhatóan beolvasott adatok mennyisége.
* A Redis-adattárak újraparticionálhatóak, de érdemes figyelembe venni, hogy ez összetett és időigényes feladat. A Redis-fürtszolgáltatás képes automatikusan újraparticionálni az adatokat, ez a funkció azonban az Azure Redis Cache-ben nem elérhető. Ezért a particionálási séma tervezésekor igyekezzen elegendő szabad helyet hagyni az egyes partíciókon az adatmennyiség idővel várható növekedésének megfelelően. Ne feledje azonban, hogy az Azure Redis Cache az adatok ideiglenes gyorsítótárazására szolgál, valamint hogy a gyorsítótárban tárolt adatok élettartama korlátozható az élettartam (TTL) érték megadásával. A viszonylag rövid életű adatok esetében az élettartam rövidre vehető, a statikus adatok esetében azonban sokkal hosszabbra. Lehetőleg ne tároljon nagy mennyiségű hosszabb élettartamú adatot a gyorsítótárban, amennyiben azok mennyisége feltehetően megtöltené a gyorsítótárat. Meghatározhat egy kiürítési szabályzatot, amely törli az adatokat az Azure Redis Cache-ből, ha a hely kiemelt fontosságú.

  > [!NOTE]
  > Az Azure Redis Cache használata esetén a gyorsítótár maximális mérete (250 MB és 53 GB között) a megfelelő tarifacsomag kiválasztásával határozható meg. Azonban a méret az Azure Redis Cache-gyorsítótár létrehozása után nem növelhető (és nem is csökkenthető).
  >
  >
* A Redis-kötegek és -tranzakciók nem terjedhetnek ki több kapcsolatra, így az egyes kötegek vagy tranzakciók által érintett összes adatot ugyanabban az adatbázisban (szegmensben) kell tárolni.

  > [!NOTE]
  > A Redis-tranzakcióban foglalt műveletszekvencia nem feltétlenül elemi jellegű. A tranzakciókat alkotó parancsokat a rendszer a futtatás előtt ellenőrzi, majd az üzenetsorba küldi azokat. Ha ebben a fázisban hiba történik, a rendszer a teljes üzenetsort elveti. Azonban miután a tranzakció sikeresen el lett küldve, az üzenetsorban lévő parancsok sorrendben lefutnak. Ha bármely parancs meghiúsul, csak az adott parancs futása áll le. A rendszer az üzenetsorban lévő összes megelőző és követő parancsot végrehajtja. További információkat a Redis webhelyén, a [tranzakciók] lapján talál.
  >
  >
* A Redis korlátozott számú elemi műveletet támogat. Az ilyen típusú műveletek közül több kulcs és érték használatát kizárólag az MGET és az MSET művelet támogatja. Az MGET műveletek kulcsok egy megadott listájához tartozó értékek gyűjteményét adják vissza, az MSET műveletek pedig ugyanezeket tárolják. Ha használni szeretné ezeket a műveleteket, az MSET és az MGET parancsok által hivatkozott kulcs-érték párokat ugyanabban az adatbázisban kell tárolnia.

## <a name="partitioning-strategies-for-azure-service-fabric"></a>Az Azure Service Fabric particionálási stratégiái
Az Azure Service Fabric egy mikroszolgáltatás-platform, amely futtatókörnyezetet biztosít az elosztott alkalmazások felhőben végzett futtatására. A Service Fabric támogatja a .Net vendégrendszer futtatható fájljait, az állapotalapú és az állapotmentes szolgáltatásokat és a tárolókat. Az állapotalapú szolgáltatások egy [megbízható gyűjteményt][service-fabric-reliable-collections] biztosítanak az adatok kulcs-érték gyűjteményben való tartós tárolásához a Service Fabric-fürtben. A kulcsok megbízható gyűjteményekben való particionálására vonatkozó stratégiákkal kapcsolatos további információkat [Irányelvek és javaslatok az Azure Service Fabric megbízható gyűjteményeihez] tartalmaznak.

### <a name="more-information"></a>További információ
* [Az Azure Service Fabric áttekintése] egy bevezető az Azure Service Fabric szolgáltatáshoz.
* [A Service Fabric Reliable Services particionálása] további információkat szolgáltat az Azure Service Fabric Reliable Services szolgáltatásairól.

## <a name="partitioning-strategies-for-azure-event-hubs"></a>Az Azure Event Hubs particionálási stratégiái

Az [Azure Event Hubs][event-hubs] nagy léptékű adatstreamelésre lett kifejlesztve, és a particionálás integrált részét képezi a horizontális skálázás lehetővé tétele érdekében. Mindegyik felhasználó az üzenetstreamnek csak egy adott partícióját olvassa. 

Az esemény-közzétevő csak a partíciókulcsot ismeri, azt a partíciót nem, amelyre az esemény közzé lesz téve. A kulcs és a partíció szétválasztása révén a küldőnek nem szükséges behatóan ismernie az alárendelt feldolgozási folyamatokat. (Eseményeket közvetlenül is küldhet adott partíciókra, az azonban általában nem ajánlott.)  

A partíciószám megadásakor hosszú távú szempontokat érdemes mérlegelni. Az eseményközpontok létrehozását követően a partíciók száma nem módosítható. 

Az Event Hubs-partíciók használatával kapcsolatos további információkat a [Mi az Event Hubs?] című cikk tartalmazza.

A rendelkezésre állás és a konzisztencia közti kompromisszummal kapcsolatos megfontolásokat a [Rendelkezésre állás és konzisztencia az Event Hubsban] című cikk tartalmazza.

## <a name="rebalancing-partitions"></a>Partíciók kiegyenlítése
Ahogy a rendszer egyre kiforrottabbá válik, és a használati minták jobban láthatóak, esetenként módosítani kell a particionálási sémát. Például előfordulhat, hogy az egyes partíciók forgalma aránytalanná válik, és kritikus pontokká válnak, ami túlzott versengést okozhat. Emellett elképzelhető, hogy alábecsülte egyes partíciók adatmennyiségét, így ezekben lassan eléri a tárhelykapacitás korlátait. Bármi legyen is az oka, néha szükségessé válik a partíciók kiegyenlítése a terhelés egyenletesebb elosztása érdekében.

Egyes adattároló rendszerek, amelyek az adatok kiszolgálókra való elosztásának módját nem hozzák nyilvánosságra, esetenként automatikusan kiegyenlíthetik a partíciókat a rendelkezésre álló erőforrások keretein belül. Más helyzetekben a kiegyenlítés egy felügyeleti feladat, amely két szakaszból áll:

1. Az új particionálási stratégia kialakítása a következők meghatározásához:
   * A felosztandó (vagy épp egyesítendő) partíciók.
   * Az adatok új partíciókra való leosztásának módja új partíciókulcsok kialakításával.
2. Az érintett adatok migrálása a régi particionálási sémáról az újonnan létrehozott partícióhalmazra.

> [!NOTE]
> Az adatbázis-gyűjtemények a kiszolgálókra való leképezése transzparens, azonban előfordulhat, hogy eléri a Cosmos DB-fiókok tárhelykapacitás- és feldolgozásisebesség-korlátait. Amennyiben ez történne, érdemes lehet átdolgoznia a particionálási sémát, és migrálnia az adatokat.
>
>

Az adattárolási módszertől és az adattároló rendszer kialakításától függően az adatok a partíciók közti migrálása esetleg azok használata közben is lehetséges (online migrálás). Ha ez nem lehetséges, szükség lehet az érintett partíciók átmeneti lekapcsolására az adatok migrálása idejére (offline migrálás).

## <a name="offline-migration"></a>Offline migrálás
Az offline migrálás valószínűleg a legegyszerűbb megközelítés, mivel ez csökkenti az esélyét annak, hogy versengés lépjen fel. Nem módosítja az adatokat, amíg azok migrálása ás átstrukturálása folyamatban van.

Elméleti szinten a folyamat a következő lépéseket foglalja magában:

1. A szegmens megjelölése offline állapotúként.
2. Az adatok felosztása/egyesítése és áthelyezése az új szegmensekre.
3. Az adatok ellenőrzése.
4. Az új szegmensek online állapotba állítása.
5. A régi szegmens eltávolítása.

A rendelkezésre állás korlátozott megőrzése érdekében az 1. lépésben az eredeti szegmens offline állapotú helyett írásvédettként is megjelölhető. Így az alkalmazások a migrálás közben is olvashatják, de nem módosíthatják az adatokat.

## <a name="online-migration"></a>Online migrálás
Az online migrálás nehezebben végrehajtható, de kevésbé zavaró a felhasználók számára, mivel az adatok a teljes folyamat során elérhetőek maradnak. A folyamat hasonlít az offline migrálás folyamatára, azzal a különbséggel, hogy az eredeti szegmenst nem kell megjelölni offline állapotúként (1. lépés). A migrálási folyamat részletességétől függően (tehát hogy a migrálás például elemenként vagy szegmensenként történik-e), az ügyfélalkalmazások kódjának adatelérésért felelős részének le kell tudnia kezelni azt, hogy az adatok olvasása és írása két helyen történik (az eredeti és az új szegmensen).

Az online migrálást támogató megoldásokra mutat példát [Skálázás az Elastic Database felosztási-egyesítési eszközének használatával] cikk a Microsoft webhelyén.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók
Az adatkonzisztencia megvalósítását célzó stratégiák kialakításakor az alábbi minták szintén relevánsak lehetnek a forgatókönyv szempontjából:

* A Microsoft webhelyének [Adatkonzisztencia – Ismertető] oldala a konzisztencia az elosztott környezetekben, például a felhőben való fenntartására vonatkozó stratégiákat ismerteti.
* A Microsoft webhelyének [adatparticionálási útmutató] oldala általános áttekintést nyújt az elosztott megoldások különféle feltételeinek megfelelő partíciók kialakításáról.
* A Microsoft webhelyén leírt [Horizontális particionálási minta] néhány gyakori stratégiát ír le az adatok horizontális particionálásához.
* A Microsoft webhelyén leírt [Indextábla minta] az adatokra vonatkozó másodlagos indexek létrehozását mutatja be. Az alkalmazás ezzel a megközelítéssel gyorsan kérheti le az adatokat olyan lekérdezések használatával, amelyek nem hivatkoznak a gyűjtemények elsődleges kulcsára.
* A Microsoft webhelyén leírt [Materializált nézet minta] az adatokat a gyors lekérdezési műveletek támogatása érdekében összegző, adatokkal előre feltöltött nézetek létrehozását írja le. Ez a megközelítés akkor lehet hasznos a particionált tárolókban, ha az összegzendő adatokat tartalmazó partíciók több helyre vannak osztva.
* A Microsoft webhelyének [Az Azure Content Delivery Network használata] foglalkozó cikke további útmutatást tartalmaz a Content Delivery Network Azure-ban való és konfigurálásához és használatához.

## <a name="more-information"></a>További információ
* A Microsoft webhely [Mi az Azure SQL Database?] című oldala részletes dokumentációval szolgál az SQL-adatbázisok létrehozásával és használatával kapcsolatban.
* A Microsoft webhely [Az Elastic Database szolgáltatásainak áttekintése] oldala átfogóan ismerteti az Elastic Database szolgáltatást.
* A Microsoft webhelyének [Skálázás az Elastic Database felosztási-egyesítési eszközének használatával] oldala a felosztás/egyesítés szolgáltatás az Elastic Database-szegmensek felügyeletéhez való használatával kapcsolatos információkat tartalmazza.
* A Microsoft webhelyének [Azure Storage skálázhatósági és teljesítménycéljait](https://msdn.microsoft.com/library/azure/dn249410.aspx) bemutató oldala az Azure Storage jelenlegi méret- és feldolgozásisebesség-korlátait dokumentálja.
* A Microsoft webhelyének [Entitáscsoport-tranzakciók végrehajtása] foglalkozó oldala részletes információkat tartalmaz a tranzakcióműveletek Azure Table Storage-ben tárolt entitásokon való végrehajtásával kapcsolatban.
* A Microsoft webhelyének [Az Azure Storage Table tervezési útmutatója] részletes információkat tartalmaz az adatok Azure Table Storage-ben való particionálásával kapcsolatban.
* A Microsoft webhelyének [Az Azure Content Delivery Network használata] foglalkozó oldala az Azure-blobtárolókban tárolt adatok Azure Content Delivery Network használatával való replikálását írja le.
* A Microsoft webhelyének [Mi az az Azure Search?] című oldala az Azure Searchben elérhető képességek teljes leírását tartalmazza.
* A Microsoft webhelyének [Az Azure Search szolgáltatási korlátozásai] foglalkozó oldala az egyes Azure Search-példányok kapacitását írja le.
* A Microsoft webhelyének [Támogatott adattípusok (Azure Search)] foglalkozó oldala a kereshető dokumentumokban és indexekben használható adattípusokat foglalja össze.
* A Microsoft webhelyének az [Azure Redis Cache] foglalkozó oldala az Azure Redis Cache bemutatását tartalmazza.
* A Redis használatával történő particionálásról információkat a Redis webhelyén, az [Particionálás: adatok felosztása több Redis-példány között] oldalán talál.
* A Microsoft webhelyének [A Redis futtatása CentOS Linux rendszerű virtuális gépen az Azure-ban] tárgyaló oldala egy példán keresztül mutatja be, hogyan hozható létre és konfigurálható egy Azure-beli virtuális gépként futó Redis-csomópont.
* A Redis webhelyének [Adattípusok] bemutató oldala a Redisben és az Azure Redis Cache-ben elérhető adattípusokat mutatja be.

[Rendelkezésre állás és konzisztencia az Event Hubsban]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Azure Redis Cache]: http://azure.microsoft.com/services/cache/
[Az Azure Storage skálázhatósági és teljesítménycéljai]: /azure/storage/storage-scalability-targets
[Az Azure Storage Table tervezési útmutatója]: /azure/storage/storage-table-design-guide
[Többnyelvű megoldások létrehozása]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Adathozzáférés nagymértékben skálázható megoldások esetén: az SQL, NoSQL és többnyelvű adatmegőrzés használata]: https://msdn.microsoft.com/library/dn271399.aspx
[Adatkonzisztencia – Ismertető]: http://aka.ms/Data-Consistency-Primer
[Adatparticionálási útmutató]: https://msdn.microsoft.com/library/dn589795.aspx
[Adattípusok]: http://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Az Elastic Database szolgáltatásainak áttekintése]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Irányelvek és javaslatok az Azure Service Fabric megbízható gyűjteményeihez]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Indextábla minta]: http://aka.ms/Index-Table-Pattern
[Materializált nézet minta]: http://aka.ms/Materialized-View-Pattern
[Többszegmenses lekérdezés]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Az Azure Service Fabric áttekintése]: /azure/service-fabric/service-fabric-overview
[A Service Fabric Reliable Services particionálása]: /azure/service-fabric/service-fabric-concepts-partitioning
[Particionálás: adatok felosztása több Redis-példány között]: http://redis.io/topics/partitioning
[Entitáscsoport-tranzakciók végrehajtása]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[Oktatóanyag Redis-fürtökhöz]: http://redis.io/topics/cluster-tutorial
[A Redis futtatása CentOS Linux rendszerű virtuális gépen az Azure-ban]: http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx
[Skálázás az Elastic Database felosztási-egyesítési eszközének használatával]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Az Azure Content Delivery Network használata]: /azure/cdn/cdn-create-new-endpoint
[Service Bus-kvóták]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Az Azure Search szolgáltatási korlátozásai]:  /azure/search/search-limits-quotas-capacity
[Horizontális particionálási minta]: http://aka.ms/Sharding-Pattern
[Támogatott adattípusok (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Tranzakciók]: http://redis.io/topics/transactions
[Mi az Event Hubs?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[Mi az az Azure Search?]: /azure/search/search-what-is-azure-search
[Mi az Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview
