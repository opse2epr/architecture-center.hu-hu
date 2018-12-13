---
title: Adatparticionálási útmutató
titleSuffix: Best practices for cloud applications
description: Útmutató a partíciók megfelelő elkülönítéséhez a független felügyelet és hozzáférés érdekében.
author: dragon119
ms.date: 11/04/2018
ms.custom: seodec18
ms.openlocfilehash: 9441c4404af991b327cd027c145604921f0223fb
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307095"
---
# <a name="horizontal-vertical-and-functional-data-partitioning"></a>Vízszintes, vertikális és funkcionális adatparticionálás

Számos nagyobb léptékű megoldásban az adatok oszlik *partíciók* , kezelhető és külön elérhető. Particionálása javíthatja a skálázhatóságot, csökkenteni a versengést, és optimalizálhatja a teljesítményt. Azt is lehetővé teszi egy elválasztó adatok mechanizmust használati minta alapján. Például archiválhatja a régebbi adatokat az adatok olcsóbb tárhelyet.

Azonban a particionálási stratégiát gondosan kell megválasztani az előnyök maximalizálása a negatív hatások minimalizálása.

> [!NOTE]
> Ebben a cikkben a *particionálás* kifejezés alatt az adatok külön adattárakba való fizikai felosztásának folyamatát értjük. Ez nem ugyanaz, mint az SQL Server táblaparticionálása.

<!-- markdownlint-disable MD026 -->

## <a name="why-partition-data"></a>Miért kell particionálni az adatokat?

<!-- markdownlint-enable MD026 -->

- **A skálázhatóság javítása**. Az önálló adatbázisrendszerek vertikális felskálázásakor a rendszer végül eléri a fizikai hardver korlátait. Ha több, egy különálló kiszolgálón lévő egyes partíciók között megoszthatjuk az adatokat horizontális felskálázása történhet a rendszer szinte végtelenül felskálázható.

- **A teljesítmény javítása**. Az egyes partíciókon az adatelérési műveletek kisebb mennyiségű adatot érintenek. Megfelelően megtörtént, a particionálás is hatékonyabb a rendszer. Az egyszerre több partíciót érintő műveletek párhuzamosan futtathatóak.

- **A biztonság javítása**. Bizonyos esetekben a különböző partíciók szét az érzékeny és a nem bizalmas adatok, és különböző biztonsági vezérlők alkalmazása a bizalmas adatok.

- **A működési rugalmasság megteremtése**. A particionálás rengeteg lehetőséget kínál a működés finomhangolására, a felügyelet hatékonyságának maximalizálására és a költségek minimalizálására. Például különböző stratégiák határozhatók meg a felügyelethez, a monitorozáshoz, a biztonsági mentéshez és helyreállításhoz, valamint egyéb felügyeleti tevékenységekhez az egyes partíciók adatainak fontossága alapján.

- **A használati mintának megfelelő adattárak használata**. A particionálás révén az egyes partíciók különböző típusú adattárakban futtathatóak az ilyen adattárak költségei és beépített szolgáltatásai alapján. Például nagy mennyiségű bináris adatok tárolhatók a blob storage-ban, míg a strukturáltabb adatot dokumentum-adatbázis tárolhatja. Lásd: [a megfelelő adattároló kiválasztása](../guide/technology-choices/data-store-overview.md).

- **A rendelkezésre állás javítása**. Az adatok több kiszolgálóra való leosztásával kiküszöbölhető az egypontos meghibásodás kockázata. Ha egy példány sikertelen, csak az adott partíció adatai nem érhető el. A többi partíción folytatódhatnak a műveletek. A felügyelt PaaS-adattárak ezt figyelembe azért kevésbé lényeges, tervezték, hogy e szolgáltatások beépített redundanciával.

## <a name="designing-partitions"></a>Partíciók tervezése

Nincsenek három tipikus adatparticionálási stratégia:

- **Horizontális particionálás** (vagy más néven *horizontális skálázás*). Ebben a stratégiában mindegyik partíció egy különálló adattárral, de az összes partíció ugyanazzal a sémával rendelkezik. Mindegyik partíció az úgynevezett egy *szegmens* és az adatokat, például az ügyfelek adott halmazának megrendeléseit egy adott részhalmazát tartalmazza.

- **Vertikális particionálás**. Ebben a stratégiában mindegyik partíció az adattárban tárolt elemek mezőinek egy alhalmazát tartalmazza. A mezők a használati mintáik alapján vannak felosztva. Például a gyakran használt mezők az egyik, a kevésbé gyakran használtak egy másik partícióba kerülhetnek.

- **Funkcionális particionálás**. Ebben a stratégiában az adatok összesítése az alapján történik, hogy a rendszer egyes körülhatárolt kontextusai hogyan használják őket. Például egy e-kereskedelmi rendszer számla adatok előfordulhat, hogy tárolása egy partícióban a termékleltárral kapcsolatos adatokat.

Ezek a stratégiák kombinálható is, és azt javasoljuk, hogy érdemes mindhármat particionálási séma kialakításakor. Például az adatokat először feloszthatja szegmensekre, majd az egyes szegmensekben lévő adatokat vertikálisan tovább particionálhatja.

### <a name="horizontal-partitioning-sharding"></a>Horizontális particionálás

1. ábra mutatja a horizontális particionálás. Ebben a példában a készletadatok termékkulcs alapján vannak szegmensekre bontva. Minden egyes szegmens a szegmenskulcsok betűrend szerint egybefüggő tartományát (A–G és H–Z) tartalmazza. A horizontális skálázás a terhelés több számítógépre, amely csökkenti a versengést és javítja a teljesítményt osztja el.

![Adatok horizontális particionálása partíciókulcs alapján](./images/data-partitioning/DataPartitioning01.png)

*1. ábra Horizontális particionálása partíciókulcs alapján (sharding) adatokat.*

A legfontosabb tényező a szegmenskulcs megfelelő megválasztása a kiválasztott. A rendszer beüzemelését követően a kulcs nehezen módosítható. A kulcsnak biztosítania kell, hogy a számítási feladatok, a lehető egyenlően oszlik el a szegmensek között az adatok particionálása.

A szegmensek nem kell az azonos méretű lehet. Több fontos, hogy a kérelmek számát egyensúlyba hozni. Lehet, hogy egyes szegmensek nagy, de minden cikk rendelkezik adatelérési műveletek száma alacsony. Más szegmensek esetleg kisebbek ugyan, de az egyes elemeket többször érik el. Fontos továbbá győződjön meg arról, hogy egyetlen szegmens nem haladja meg a skálázási korlátait (kapacitás és feldolgozási erőforrások) tekintetében az adattár.

Ne hozzon létre, amely hatással lehet a teljesítmény és rendelkezésre állás "Forró" partíciókat. Például a felhasználó nevének első betűje rendelkezik egy okozta egyenetlen eloszlást, mivel bizonyos betűk gyakori Instead, ügyfél-azonosító kivonata segítségével adatokat egyenletesebben osszák partíciók között.

Válasszon olyan szegmenskulcsot, amely minimálisra csökkenti a nagy szegmenseket fel, a kisebbeket nagyobbakká egyesíteni, vagy a séma módosításához esetleges jövőbeli követelményeket. Az ilyen műveletek nagyon időigényesek lehetnek, és a végrehajtásuk során esetleg néhány szegmenst le is kell állítani.

Ha a szegmensek replikálva vannak, a replikák némelyike esetleg online tartható, amíg a többi felosztása, egyesítése vagy átkonfigurálása folyik. A rendszer azonban szükség lehet az újrakonfigurálás alatt végrehajtható műveletek korlátozására. Ha például a replikákon lévő adatok jelöléssel csak olvasható adatok inconsistences elkerülése érdekében.

További információ a horizontális particionálási, lásd: [horizontális skálázási minta].

### <a name="vertical-partitioning"></a>Vertikális particionálás

A vertikális particionálás tevékenység leggyakoribb felhasználási csökkenteni az i/o és gyakran használt elemek beolvasása, amely teljesítmény költségeket. A 2. ábrán a vertikális particionálás egy példája látható. Ebben a példában egy elem különböző tulajdonságai különböző partíciókon vannak tárolva. Egy partíció elért adatokat gyakrabban, beleértve a termék nevét, leírását és árát tartalmazza. Egy másik partíció tartalmazza a leltáradatok: a leltár és az utolsó rendezett dátum.

![Adatok vertikális particionálása a használati minta alapján](./images/data-partitioning/DataPartitioning02.png)

*2. ábra Adatok vertikális particionálása a használati minta alapján.*

Ebben a példában az alkalmazás rendszeresen lekérdezi a termékek nevét, leírását és árát, amikor megjeleníti a termékek részleteit az ügyfelek számára. Leltár és az utolsó – dátum rendezett egy külön partíciót, mert ezt a két elemet általában együtt használják tartanak.

A vertikális particionálás többi előnyei:

- Viszonylag adatokat (a termék nevét, leírását és árát) is el kell különíteni a dinamikus adatok (mennyiségektől és az utolsó megrendelések dátumától). Lassú helyez át adatokat egy jó jelölt az alkalmazás számára a memóriában lévő gyorsítótárhoz.

- Bizalmas adatok tárolhatók egy külön partíció a további biztonsági ellenőrzéseket.

- A vertikális particionálás csökkentheti az egyidejű elérésének mennyisége.

A vertikális particionálás az entitások szintjén működik a tárolóban, részlegesen normalizálja az entitásokat, és *széles* elemekből *keskeny* elemekké bontja le azokat. Kiválóan alkalmazható oszlopalapú, például HBase és Cassandra adattárakhoz. Ha az oszlopok egy gyűjteményében lévő adatok valószínűleg nem változnak majd, megpróbálhat oszloptárolókat is használni az SQL Serverben.

### <a name="functional-partitioning"></a>Funkcionális particionálás

Amikor azonosítható egy körülhatárolt kontextus az alkalmazások minden egyes külön üzleti területhez, funkcionális particionálást módja a elkülönítés és az adatelérési teljesítmény javításához. A funkcionális particionálás egy másik gyakori alkalmazási az írható-olvasható adatok elkülönítése a csak olvasható adatok. A 3. ábrán a funkcionális particionálás áttekintése látható egy példán, ahol a készletadatok elkülönülnek az ügyféladatoktól.

![Adatok funkcionális particionálása körülhatárolt kontextus vagy részterület alapján](./images/data-partitioning/DataPartitioning03.png)

*3. ábra Funkcionális particionálása körülhatárolt kontextus vagy részterület alapján adatokat.*

Ez a particionálási stratégia csökkentheti az adatelérési versengést a rendszer egyes részei között.

## <a name="designing-partitions-for-scalability"></a>Partíciók tervezése skálázhatóságra

Elengedhetetlen az egyes partíciók méretének és munkaterhelésének mérlegelése és kiegyensúlyozása, hogy az adatok megfelelően legyenek elosztva a maximális skálázhatóság érdekében. Azonban az adatok particionálása során arra is figyelni kell, hogy ne haladják meg az egypartíciós tárolók skálázási korlátait.

A partíciók skálázhatóságra tervezése során kövesse az alábbi lépéseket:

1. Az alkalmazás elemzésével ismerje meg a hozzáférési mintákat, például az egyes lekérdezések által visszaadott eredményhalmazok méretét, az adatelérési gyakoriságot, az eredendő késést és a kiszolgálóoldali számítási feldolgozás követelményeit. Sok esetben néhány nagyobb entitás igényli a szükséges erőforrások többségét.
2. Az elemzés segítségével határozza meg az aktuális és a jövőbeli skálázhatósági célokat, például az adatok méretét és a munkaterhelést. Ezután ossza el az adatokat a partíciók közt a skálázhatósági célok teljesítéséhez. A horizontális particionálás választja, a megfelelő szegmenskulcs fontos, hogy győződjön meg arról, hogy terjesztési páros. További információkért tekintse meg a [horizontális skálázási minta].
3. Győződjön meg arról, hogy mindegyik partíció nem rendelkezik elegendő erőforrással adatméret és a teljesítmény, a skálázhatósági követelmények kezelésére. Az adattár függően előfordulhat tárolóhely feldolgozási teljesítményt vagy a hálózati sávszélesség partíciónként mennyisége korlátozva. Ha a követelmények valószínűleg meghaladják majd ezeket a korlátokat, szükség lehet a particionálási stratégia finomítására vagy az adatok további, felosztására valószínűleg a legalább két stratégia kombinálása.
4. Győződjön meg arról, hogy az adatok megfelelően vannak-e, és, hogy a partíciók képesek kezelni a terhelést a rendszer monitorozásával. Tényleges használat nem mindig egyezik az elemzés előrejelzi. Ebben az esetben újra kiegyensúlyozhatók a partíciók, vagy pedig újra kell terveznie a rendszert, hogy a szükséges egyensúly bizonyos részeit is lehet.

Néhány foglalják le az erőforrásokat infrastruktúra határai tekintetében. Bizonyosodjon meg róla, hogy a kiválasztott határ korlátai elegendő teret biztosítanak az adatmennyiség várható növekedéséhez az adattárolás, a feldolgozási teljesítmény és a sávszélesség tekintetében.

Például ha az Azure table storage szolgáltatást használja, nincs egy partíció által egy adott időszakban feldolgozható kérelmek mennyisége korlátozva. (Lásd: [Az Azure Storage skálázhatósági és teljesítménycéljai].) Foglalt szegmensek több erőforrást, mint egy partíció képes kezelni lehet szükség. Ha igen, a szegmenst esetleg újra kell kell particionálni a terhelés elosztását. Ha a teljes mérete vagy a táblák átviteli meghaladja a tárfiók kapacitását, szükség lehet további tárfiókokat létrehozni, és elosztani a táblákat a fiókok közt.

## <a name="designing-partitions-for-query-performance"></a>Partíciók tervezése lekérdezési teljesítményre

A lekérdezési teljesítmény gyakran kisebb adatkészletek használatával és a lekérdezések párhuzamos futtatásával növelhető. Mindegyik partíciónak a teljes adatkészlet egy kis részét kell tartalmaznia. A mennyiség csökkenésével javulhat a lekérdezések teljesítménye. A particionálás azonban nem váltja ki az adatbázis megfelelő kialakítását és konfigurálását. Például győződjön meg arról, hogy már működik-e a szükséges indexeket a.

A partíciók lekérdezési teljesítményre tervezése során kövesse az alábbi lépéseket:

1. Vizsgálja meg az alkalmazás követelményeit és teljesítményét:

   - Üzleti követelmények segítségével határozhatja meg a kritikus lekérdezéseket, amelyek mindig gyorsan kell lefutnia.
   - A rendszer monitorozásával azonosítsa a mindig lassan lefutó lekérdezéseket.
   - Keresse meg, mely lekérdezések leggyakrabban történik. Akkor is, ha egyetlen lekérdezést egy minimális költségű rendelkezik, akkor az összesített erőforrás-használat jelentős hatással lehet.

2. Particionálja a teljesítményt csökkentő adatokat:
   - Korlátozza az egyes partíciók méretét, hogy a lekérdezések válaszideje a célon belül maradjon.
   - Horizontális particionálás használatakor tervezze a szegmenskulcs úgy, hogy az alkalmazás egyszerűen kiválaszthatja a megfelelő partíció. Így a lekérdezésnek nem kell az összes partíciót átvizsgálnia.
   - Mérlegelje a partíciók elhelyezését. Ha lehetséges, próbálja az adatokat olyan partíciókban tárolni, amelyek földrajzilag közel helyezkednek el az adatokhoz hozzáférő alkalmazásokhoz és felhasználókhoz.

3. Ha egy entitás a feldolgozási sebességre vagy lekérdezési teljesítményre vonatkozó követelményekkel rendelkezik, alkalmazzon funkcionális particionálást az adott entitás alapján. Ha ez még mindig nem elégíti ki a követelményeket, alkalmazzon horizontális particionálást is. A legtöbb esetben egyetlen particionálási stratégia elegendő lesz, egyes esetekben azonban a két stratégia kombinálása hatékonyabbnak bizonyul.

4. Vegye figyelembe, hogy fut a lekérdezések párhuzamos partíciókon a teljesítmény javítása.

## <a name="designing-partitions-for-availability"></a>Partíciók tervezése rendelkezésre állásra

Az adatok particionálásával javítható az alkalmazások rendelkezésre állása, mivel így a teljes adatkészlet nem jelent egypontos meghibásodási helyet, és az adatkészlet egyes részhalmazai egymástól függetlenül kezelhetők.

Vegye figyelembe a rendelkezésre állást befolyásoló alábbi tényezőket:

**Mennyire kritikusak az adatok az üzleti tevékenységek szempontjából**. Azonosítani, hogy mely adatok kritikus fontosságú üzleti adatokat, például tranzakciók, és mely adatok kevésbé fontos működési adatok, például a rendszernapló fájljaiban.

- Vegye figyelembe, hogy kritikus fontosságú adatokat tárolja magas rendelkezésre állású partíciókon, megfelelő biztonsági mentési tervvel.

- Külön felügyeleti és figyelési eljárások a különböző adatkészleteket hoz létre.

- Az ugyanolyan kritikusságú adatokat helyezze ugyanabba a partícióba, hogy megfelelő gyakorisággal egyszerre készíthessen biztonsági másolatot róluk. Például tranzakciók adatait tároló igényelhet, mint a naplózási és nyomkövetési információkat tartalmazóakról gyakrabban kell készíteni.

**Hogyan felügyelhetőek az egyes partíciók**. A partíciók a független felügyeletet és karbantartást biztosító kialakítása több előnnyel is jár. Példa:

- Ha valamelyik partíció leáll, önállóan helyreállítható, anélkül, hogy a többi partíció adatait használó alkalmazások.

- Az adatok földrajzi területek szerinti particionálásával az ütemezett karbantartás minden helyen a csúcsidőn kívül végezhető el. Gondoskodjon róla, hogy a partíciók ne legyenek túl nagyok a tervezett karbantartás adott időablakban való elvégzéséhez.

**Érdemes-e a kritikus fontosságú adatokat a partíciók közt replikálni**. Ez a stratégia javíthatja a rendelkezésre állást és teljesítményt, de adatkonzisztenciával kapcsolatos problémák is eredményezhet. Szinkronizálni a módosításokat az összes replikára időt vesz igénybe. Ez alatt az idő alatt az adatértékek a különböző partíciókban különbözőek lesznek.

## <a name="application-design-considerations"></a>Alkalmazáskialakítási szempontok

A particionálás bonyolultabbá teszi tervezését és fejlesztését, a rendszer. A particionálást a rendszer alapvető részeként kell tekinteni a tervezés során, még ha a rendszer eleinte csak egyetlen partícióval rendelkezik is. Ha a particionálást csak utólag, nehéz lehet, mert már rendelkezik egy élő rendszert karban kell tartani lesz:

- Adatelérési logikáját kell módosítani.
- Előfordulhat, hogy nagy mennyiségű meglévő adatot kell telepíthetők át, a partíciók közötti
- Felhasználók elvárják, hogy továbbra is használható a rendszer az áttelepítés során.

Esetenként a particionálást nem tekintik fontosnak, mivel kezdetben még kisméretű az adatkészlet, és egy kiszolgáló könnyen képes kezelni azt. Ez lehet bizonyos számítási feladatokhoz igaz, de a felhasználók számának növekedésével növekednie kell számos kereskedelmi rendszer.

Továbbá már nem csak a particionálás nagy mennyiségű adat tárolja. Például egy kisméretű adattárnak is lehet intenzív forgalma több száz egyidejű ügyfél irányában. Ilyen esetben az adatok particionálása segíthet csökkenteni a versengést és növelni a teljesítményt.

Az adatparticionálási sémák kialakításakor vegye figyelembe a következő szempontokat:

**Több partíciót érintő adatelérési műveletek minimalizálása**. Ahol lehetséges, a leggyakrabban használt adatbázis-műveletek adatokat tartsa együtt az egyes partíciókban a több partíciót érintő adatelérési műveletek minimalizálása érdekében. Több partícióra kiterjedő lekérdezések több időt vesz igénybe, mint az ugyanazon a partíción belül is lehet, de a partíciók a lekérdezések optimalizálása kedvezőtlen hatással lehet más készletnyi lekérdezést. Ha le kell kérdezni a partíciók közt, minimálisra csökkenthető a lekérdezési idő lekérdezések párhuzamos futtatásával, és összesíti az eredményeket az alkalmazáson belül. (Ez a módszer nem lehetséges, bizonyos esetekben, például ha egy lekérdezés eredménye a következő lekérdezés lesz használva.)

**Vegye figyelembe, statikus referenciaadatok replikálását.** Ha a lekérdezések viszonylag statikus referenciaadatokra vonatkoznak, mint például az irányítószámok táblázatára vagy terméklistákra, használjon céltárfiókban ezeket az adatokat az összes partícióra külön keresési műveletet végezni a különböző partíciókban csökkentése érdekében. Ez a megközelítés a referenciaadatok olyan "Forró" adatkészlet, a teljes rendszer között a nagy forgalmú váljon valószínűségét is csökkentheti. Van azonban egy esetleges szinkronizálása a referenciaadatok további költségekkel.

**A partíciók közti összekapcsolásokat minimalizálása érdekében.** Ahol lehetséges, minimalizálja a hivatkozásintegritás-vertikális és funkcionális partíciókban között. Ezekben a sémákban az alkalmazás feladata a hivatkozásintegritás fenntartása a partíciók között. Adatok join több több partícióra kiterjedő lekérdezések nem hatékony, mert az alkalmazás általában egy kulcsot és egy külső kulcs alapján egymást követő lekérdezéseket kell. Ehelyett érdemes replikálni a vonatkozó adatokat vagy megszüntetni azok normalizáltságát. Ha a partíciók közti összekapcsolásokat szükség, futtassa párhuzamosan a lekérdezéseket a külön partíciókban, és az adatokat az alkalmazáson belül.

**Végső konzisztencia támogatása**. Mérje fel, hogy az erős konzisztencia valóban követelmény-e. Általánosan használt megközelítés elosztott rendszerek, hogy a végleges konzisztencia megvalósításával. Az egyes partíciók adatainak frissítése külön történik, és az alkalmazáslogika biztosítja, hogy a frissítések mind sikeresen mennek végbe. A logika emellett a végül konzisztens műveletek keretében futtatott adatlekérdezésekből eredő esetleges inkonzisztenciákat is kezeli.

**Mérlegelje, hogyan találják meg a lekérdezések a megfelelő partíciót**. Ha egy lekérdezésnek az összes partíciót át kell vizsgálnia a szükséges adatok megtalálásához, az jelentős mértékben kihathat a teljesítményre, még akkor is, ha több párhuzamos lekérdezés fut. A vertikális és funkcionális particionálás, a lekérdezések természetes módon képesek meghatározni a partíciót. Horizontális particionálás, másrészt teheti nehéz, elemek helyének megkeresése, mert minden szegmens ugyanazzal a sémával. Egy tipikus megoldás, amellyel keresse meg az egyes adatelemek helyét a szegmensek közt leképezést fenntartani. Ez a térkép az alkalmazás horizontális particionálási logikájában valósítható meg, vagy az adattár is karbantarthatja, ha támogatja a transzparens horizontális particionálást.

**Vegye figyelembe, rendszeres időközönként a szegmensek újraegyensúlyozása**. Horizontális particionálás, a szegmensek újraegyensúlyozása segíthet az adatok egyenletes elosztását méret és minimalizálja a kritikus pontok, a lekérdezési teljesítmény maximalizálása és a fizikai megkerüléséhez. Ez azonban egy összetett feladat, amelyhez általában egy egyéni eszköz vagy folyamat használata szükséges.

**A partíciók replikálása.** Ha mindegyik partíciót replikálja, ez meghibásodása elleni további védelmet biztosít. Ha valamelyik replika leáll, a lekérdezések egy másik, működő példányra irányíthatóak.

**Ha eléri valamely particionálási stratégia fizikai korlátait, érdemes lehet a skálázhatóságot egy másik szintre kiterjeszteni**. Például ha a particionálás az adatbázis szintjén valósul meg, a partíciókat esetleg több adatbázisban szükséges elhelyezni vagy replikálni. Ha a particionálás már eleve az adatbázis szintjén van megvalósítva, és a fizikai korlátok problémát jelentenek, ez jelezheti azt, hogy a partíciókat esetleg több üzemeltetési fiókban szükséges elhelyezni vagy replikálni.

**Kerülje a több partícióban lévő adatokat használó tranzakciókat**. Egyes adattárak tranzakció-konzisztenciát és integritást valósítanak meg az adatokat módosító műveletekre, de csak abban az esetben, ha az adatok egyetlen partíción találhatóak. Ha több partícióra kiterjedő tranzakciótámogatás szükséges, ezt valószínűleg az alkalmazáslogika részeként kell megvalósítania, mivel a legtöbb particionálási rendszer ezt natív módon nem támogatja.

Minden adattárhoz bizonyos üzemeltetési felügyeleti és monitorozási tevékenységeket is végezni kell. Ilyen feladat lehet az adatok betöltése, az adatok biztonsági mentése és helyreállítása, az adatok átrendezése, valamint a rendszer megfelelő és hatékony működésének biztosítása.

Mérlegelje az üzemeltetési felügyeletet befolyásoló alábbi tényezőket:

- **Hogyan valósíthatóak meg a megfelelő felügyeleti és üzemeltetési feladatok az adatok particionálása esetén**. Ilyen feladatok lehetnek a biztonsági mentés és visszaállítás, az adatok archiválása, a rendszer monitorozása, valamint egyéb felügyeleti feladatok. Például a logikai konzisztencia fenntartása a biztonsági mentési és visszaállítási műveletek során kihívást jelenthet.

- **Hogyan tölthetőek be az adatok több partícióba, és hogyan adhatók hozzá a külső forrásokból érkező új adatok**. Egyes eszközök és segédprogramok esetleg nem támogatják a horizontálisan particionált adatműveleteket, például az adatok a megfelelő partícióba való betöltését.

- **Hogyan oldható meg az adatok rendszeres archiválása és törlése**. A partíciók méretének túlzott növekedését elkerülendő az adatokat rendszeresen (például havonta) archiválni és törölni kell. Szükség lehet az adatok egy másik archiválási sémának megfelelő átalakítására.

- **Hogyan találhatók meg az adatintegritási problémák**. Fontolja meg, keresse meg az adatintegritási problémák, például egy olyan partíciót, egy másik hiányzó adataira hivatkoznak az adatok egy rendszeres folyamat futtatását. A folyamat megpróbálhatja automatikusan megoldhatja ezeket a problémákat, vagy egyszerűen csak a manuális ellenőrzést jelentés létrehozása.

## <a name="rebalancing-partitions"></a>Partíciók kiegyenlítése

A rendszer kiforrottá, előfordulhat, hogy kell particionálási sémát módosítani. Például előfordulhat, hogy az egyes partíciók indítson el egy aránytalanul nagy mennyiségű forgalmat és a válnak a gyakori elérésű, ami túlzott versengést okozhat. Vagy, előfordulhat, hogy rendelkezik alábecsülte egyes partíciók adatmennyiség okozza az egyes partíciók kapacitáskorlátait megközelítést.

Néhány adattár, például a Cosmos dB-ben is automatikusan egyensúlyozni a partíciókat. Más esetekben újraegyensúlyozása egy felügyeleti feladat, amely két szakaszból áll:

1. Egy új particionálási stratégia meghatározása.

    - Felosztandó (vagy épp egyesítendő) partíciók kell?
    - Mi az az új partíciós kulcs?

2. Adatokat át a régi particionálási sémáról az újonnan létrehozott partícióhalmazra.

Az adattár függően előfordulhat, hogy kell adatok áttelepítése közben használatban vannak a partíciók között. Ezt nevezzük *online migrálás*. Ha ez nem lehetséges, szüksége lehet győződjön meg arról, partíciók nem érhető el, amíg az adatok áthelyezéséről van (*offline migrálás*).

### <a name="offline-migration"></a>Offline migrálás

Offline áttelepítés csökkenti a versengést előfordulásának esélyét oka általában egyszerűbb. Elméleti szinten offline áttelepítés a következőképpen működik:

1. A partíció offline megjelölni.
2. Felosztási-egyesítési, és áthelyezi az adatokat az új partíciókat.
3. Az adatok ellenőrzése.
4. Az új adatpartícióinak online állapotba.
5. Távolítsa el a régi partíció.

Igény szerint jelölheti egy partíció 1. lépésben csak olvasásra, hogy az alkalmazások továbbra is képes olvasni az adatokat, közben is olvashatják.

## <a name="online-migration"></a>Online migrálás

Online migrálás nehezebben végrehajtható, de kevésbé zavaró. A folyamat hasonlít az offline migrálás, kivéve az eredeti partíció nincs megjelölve offline állapotban van. (Például elem és a szegmens szegmens szerint) a migrálási folyamat részletességétől függően az adatelérési kód az ügyfélalkalmazások lehet olvasása és írása két helyen, az eredeti partíció, és az új a tárolt adatok kezelése a partíció.

## <a name="related-patterns"></a>Kapcsolódó minták

Az alábbi tervezési minták a forgatókönyvéhez hasznosak lehetnek:

- A [horizontális skálázási minta](../patterns/sharding.md) az adatok horizontális néhány gyakori stratégiát ismerteti.

- A [indextáblát alkalmazó minta](../patterns/index-table.md) adatokra vonatkozó másodlagos indexek létrehozását mutatja be. Az alkalmazás ezzel a megközelítéssel gyorsan kérheti le az adatokat olyan lekérdezések használatával, amelyek nem hivatkoznak a gyűjtemények elsődleges kulcsára.

- A [materialized view minta](../patterns/materialized-view.md) ismerteti, hogyan hozhat létre, amely a gyors lekérdezési műveletek támogatása az adatok összegzéséhez adatokkal előre feltöltött nézetek. Ez a megközelítés akkor lehet hasznos a particionált tárolókban, ha az összegzendő adatokat tartalmazó partíciók több helyre vannak osztva.

## <a name="next-steps"></a>További lépések

- További információ a particionálási stratégiái az adott Azure-szolgáltatásokhoz. Lásd: [adatok particionálási stratégiái](./data-partitioning-strategies.md)

[Az Azure Storage skálázhatósági és teljesítménycéljai]: /azure/storage/storage-scalability-targets
