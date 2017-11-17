---
title: "Esemény forrás"
description: "Egy csak append tároló segítségével rögzítheti a teljes eseménysorozatokat, amelyek egy tartományban lévő adatokon műveleteit tartalmazzák."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: d5d4e99a6ff49cb823f592c83590471c0d68bfd1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="event-sourcing-pattern"></a>Esemény Sourcing minta

[!INCLUDE [header](../_includes/header.md)]

Helyett csak az adatok aktuális állapotát egy tartományban, használja a egy csak append tároló műveleteket végezhet el az adatok a teljes sorozatát rögzítéséhez.
A tároló úgy működik, mint a rendszer rekord, és a tartomány objektumok materializálni használható. Ezzel egyszerűsítheti feladatok összetett tartományok, így nem kell szinkronizálni az adatokat az adatmodellbe és a vállalati tartományhoz, teljesítmény, méretezhetőség és reakcióidőt fokozása mellett. Azt is adja meg a konzisztencia tranzakciós adatok, és teljes naplózási előzmények kezelése és engedélyezheti a kompenzációs műveletek előzményeinek megőrzése.

## <a name="context-and-problem"></a>A környezetben, és probléma

A legtöbb alkalmazás adatokkal dolgozni, és a szokásos megoldás, az alkalmazás az adatok aktuális állapotának karbantartásához frissítené azt a felhasználók használható. Például a hagyományos létrehozás, Olvasás, frissítés és Törlés (CRUD) modell a jellemző adatok folyamat az adatokat olvasni az áruházból, a módosításokat bizonyos azt, és az adatok aktuális állapotának frissítéséhez az új értékekkel&mdash;gyakran tranzakciók használatával az adatok, amelyek zárolása.

A CRUD megközelítés azzal bizonyos korlátozások vonatkoznak:

- CRUD rendszerek hajtsa végre a frissítési műveleteket közvetlenül a tárolóban, így lelassíthatják a teljesítmény és reakcióidőt, és korlátozza a méretezhetőség miatt a feldolgozási terhelést van szükség.

- Olyan együttműködési tartomány nagyszámú egyidejű felhasználót frissítés adatütközések valószínűbb, mert a frissítési műveletek csak egy elemet az adatok alapján történik.

- Kivéve, ha egy további naplózási mechanizmus, amely az egyes műveletek részleteit rögzíti egy külön naplóban, el fog veszni.

> Bemutatják a CRUD megközelítés határain, lásd: [CRUD, csak ha akkor is biztosít az](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).

## <a name="solution"></a>Megoldás

Az esemény forrás mintát megközelítés a kezelési műveletek, amelyek célja a események, sorozata, amelyek csak a hozzáfűző tárolóban rögzítése adatokon határozza meg. Alkalmazáskód küld egy eseménysorozatokat, amelyek imperatively ismertetik, hogy történt az esemény tárolóba, ahol éppen megőrzött adatokat. Minden esemény módosítások készletét reprezentálja, az adatok (például `AddedItemToOrder`).

Az események megmaradnak, amely a rekord (a mérvadó adatforrása) az adatok aktuális állapotát a rendszer esemény tárolóban. Az esemény tároló általában ezeket az eseményeket tesz közzé, hogy a felhasználók értesítést kapni, és kezelheti azokat szükség esetén. Fogyasztók nem, például más rendszerek az események műveletek feladatokat kezdeményez, vagy hajtsa végre az egyéb kapcsolódó művelet, amely a művelet végrehajtásához szükséges. Figyelje meg, hogy az alkalmazás kódjában, amely létrehozza az eseményeket a rendszer leválasztja a rendszerekre, amelyek az események előfizetni.

Az események az esemény tároló által közzétett a gyakori felhasználási entitások materializált nézetek fenntartásához műveletek az alkalmazás módosítsa őket, és külső rendszerekkel való integrációra. Például a rendszer fenntarthatja a materializált nézet összes ügyfél rendelés, amely a felhasználói felület részei feltöltésére használt eszköz. Az alkalmazás új rendelések hozzáadja, hozzáadja vagy eltávolítja az elemet a sorrendnek, és hozzáadja a szállítási adatokat, a módosítások leíró események kezelje-e, és hogy frissítheti a [materializált nézet](materialized-view.md).

Emellett bármikor lehetőség az alkalmazások számára az előzmények események olvasását, és vissza és adott entitáshoz kapcsolódó összes eseményt fel entitás jelenlegi állapotának materializálni segítségével. Ez akkor fordulhat elő, a tartományobjektum materializálni, amikor a kérelmet kezelő igény szerint vagy ütemezett feladat keresztül, hogy az entitás állapotát is tárolhatók a bemutató réteg támogatásához materializált nézetként.

Az ábra mutatja a mintát, köztük egyes a beállításokat, például a materializált nézetek létrehozásával, külső alkalmazások és rendszerek események integrálása és események visszajátszását az eseménystream használatával történő létrehozásához leképezések aktuális állapotának áttekintése adott entitások.

![Áttekintés és példa az esemény forrás minta](./_images/event-sourcing-overview.png)


Az esemény forrás mintát a következő előnyöket nyújtja:

Események nem módosíthatók, és csak a hozzáfűző művelet is tárolhatók. A felhasználói felület, a munkafolyamat vagy a folyamat, amely egy esemény kezdeményezett továbbra is, és feladatokat, amelyek az események kezelésére is fusson a háttérben. Kombinálva a tényt, hogy van-e nem a versengés tranzakciók, a feldolgozás során jelentősen javítja a teljesítmény és méretezhetőség, az alkalmazások, különösen a bemutatási szint vagy a felhasználói felület.

Események olyan egyszerű-objektumok leíró néhány művelet, amely történt, és minden kapcsolódó adatairól, hogy az esemény által képviselt művelet leírása. Események közvetlenül nem frissíti a tárolóban. Azok egyszerűen megfelelő időben kezelésére van rögzítve. Ez egyszerűbbé teszi megvalósítása és kezelése.

Események általában jelentéssel bírnak az egy tartomány szakértő, mivel [objektum relációs impedancia eltérés](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) összetett adatbázistáblák megértése nehéz tehet. Táblák olyan mesterséges szerkezetek, amelyek megfelelnek a rendszer, a jelenlegi állapotában nem előforduló eseményeket.

Esemény forrás segítségével megakadályozhatja, hogy a párhuzamos frissítések ütközést okoz, mivel ezzel elkerülheti a követelmény közvetlenül objektumokat frissíteni az adattárban. Azonban a modell továbbra is kell megtervezni, a kérelmek, ami okozhatja azt inkonzisztens állapotban védeni magát.

Az események csak append tárolását biztosít, amely által végrehajtott műveletek elleni adattárat, újragenerálása a jelenlegi állapotában materializált nézetek vagy leképezések az események visszajátszását bármikor figyelésére használható, és annak elősegítéséhez, a tesztelés és hibakeresés elővizsgálati a a rendszer. A vonatkozó követelményt a változtatások visszavonásához kompenzációs eseményeknek a segítségével emellett egy korábbi módosításait, amely lettek visszaállítva, amely így akkor fordulhat elő, ha a modell egyszerűen a jelenlegi állapotában. Az események listájának is használható, az alkalmazások teljesítményének elemzése és a felhasználói viselkedés trendek észlelése, vagy más hasznos üzleti adatok beszerzéséhez.

Az esemény tároló eseményeket indít, és a feladatok műveleteket ezeket az eseményeket adott válaszként. Ez a feladatok az események leválasztásával biztosít rugalmasságot és bővíthetőséget. Feladatok tudja, milyen típusú eseményt és eseményadatok, de nem az eseményt kiváltó működésével kapcsolatos. Több feladat emellett képes kezelni minden esemény. Ez lehetővé teszi, hogy az egyszerű integráció az egyéb szolgáltatások és rendszerekről, amelyek csak az esemény tároló által kiváltott új események figyelését. Azonban az esemény forrás események általában nagyon alacsony, és helyette az adott integrációs események generálásához szükség lehet.

> Esemény forrás általában együtt a CQRS minta az adatok kezelési feladatainak végrehajtása az eseményekre válaszként, valamint a tárolt események nézeteket materializálása.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

A rendszer csak akkor idővel konzisztenssé, ha létrehozása materializált nézet, és az adatok előállítása leképezések által események visszajátszását. Egyes események az esemény tanúsítványtárolójához adását eredményezi egy kérést a közzétett események kezelésére kérelmet, és azokat az eseményeket a késleltetés van. Ebben az időszakban, leíró további módosítások entitások új események előfordulhat, hogy az esemény üzletben érkezett.

> [!NOTE]
> Tekintse meg a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx) végleges konzisztencia kapcsolatos információkat.

Az esemény tároló információk állandó forrását, és így eseményadatok soha nem kell frissíteni. A csak a módosítások visszavonásához entitás frissítése, kompenzációs esemény hozzáadása az esemény-tárolóhoz. Ha a formátum (az adatok helyett) a megőrzött események módosítani kell, lehet, hogy a áttelepítés során nehézkes lehet egyesítése meglévő események a tárolóban lévő új verzióval. Módosítások végrehajtása úgy is kompatibilis az új formátummal összes esemény iterációt, vagy adja hozzá az új formátumot használó új eseményeket szükség lehet. Érdemes lehet egy verziószámmal minden az esemény-séma verziója is a régi és az új esemény-formátumok.

Többszálas alkalmazások és az alkalmazások több példánya lehet, hogy tárolni események az esemény-tárolóban. Események az esemény tárolójában konzisztencia létfontosságú, mivel egy adott entitás (a sorrendet, hogy a változás entitás hatással van a jelenlegi állapotában) érintő események sorrendjének. Időbélyeg ad hozzá minden esemény segíthet problémák elkerülése érdekében. Másik gyakori eljárása minden eredő egy kérelem növekményes azonosítójú esemény a jegyzet. Ha két művelet megkísérli eseményeket ugyanaz az entitás hozzáadása egy időben, az esemény tároló elutasíthatja egy eseményt, amely megegyezik egy meglévő entitásazonosító és eseményazonosító.

Nincs szabványos módszert, vagy az SQL-lekérdezések, például a meglévő mechanizmusok tájékoztatást kaphat az események olvasását. Az adatok közül kizárólag kiolvasható áll eseményeket esemény azonosító feltételt. Egyes szervezetek általában összekapcsolódnak eseményazonosító. A jelenlegi állapotában az entitás csak meghatározható visszajátszását az események, amelyek kapcsolódnak az ellen, hogy az entitás eredeti állapotát.

Minden esemény adatfolyam hossza kezelése és a rendszer frissítése hatással van. Ha az adatfolyamokat nagy, fontolja meg a megadott számú események például adott időközönként pillanatképeinek. Az entitás jelenlegi állapotában érhető el a pillanatképből, és később idő előfordult eseményeket visszajátszását. Pillanatképeket készíteni létrehozásával kapcsolatos további információkért lásd: [pillanatkép-Anna Fowler vállalati alkalmazás architektúra webhelyen](http://martinfowler.com/eaaDev/Snapshot.html) és [fő-alárendelt pillanatkép-replikálási](https://msdn.microsoft.com/library/ff650012.aspx).

Annak ellenére, hogy az esemény forrás minimalizálja az esélye, az adatok ütköző frissítések, az alkalmazás továbbra is képesnek kell lennie inkonzisztenciát foglalkozik a végleges konzisztencia származó és a tranzakciók hiánya. Például tőzsdei készlet csökkenését jelző esemény előfordulhat, hogy az ügyfélszámítógépekre érkeznek az adattárban egy sorrendet, hogy az elem alatt helyezkedik el, a követelmény, hogy aztán összeegyeztesse a két művelet, vagy az ügyfél tanácsadás, vagy hozzon létre egy rendelés eredményezve közben.

Esemény kiadvány "legalább egyszeri" lehet, és így az események fogyasztók idempotent kell lennie. A frissítés ismertetett egy eseményt, ha az esemény kezelése egynél többször nem azokat kell telepítenie. Például ha egy felhasználó több példánya karbantartása összesítő egy entitás tulajdonság, például a megrendelések, teljes száma csak az egyiket kell sikeres összesített növekvő sorrendhez elhelyezett esemény bekövetkezésekor. Bár ez nem egy forrás esemény kulcsfontosságú jellemzője, ez a beállítás a szokásos megvalósítási döntési.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta a következő helyzetekben használhatja:

- Ha szeretné rögzíteni a célt, cél vagy az adatok OK. Például egy ügyfélentitást módosításai rögzíthetők adott eseménytípusok sorozataként például _áthelyezett otthoni_, _lezárt számla_, vagy _elhunyt_.

- Ha elengedhetetlen, hogy minimalizálja vagy teljesen megelőzze adatok ütköző frissítéseit.

- Ha azt szeretné, és lehet, hogy a rendszer állapotának visszaállítása, visszaállítható a változás, vagy egy megőrzése, és eseménynaplója visszajátszásos eseményeket rögzíteni. Például ha egy feladat több lépésből áll szükség lehet visszaállítani a frissítések és majd az egyes lépéseket annak érdekében, az adatok ismét konzisztens állapotának visszajátszásos célzó műveletek végrehajtásához.

- Események használata esetén az alkalmazás működésének természetes szolgáltatása, és kissé további fejlesztési vagy megvalósítási beavatkozást igényel.

- Ha szeretné absztrakcióhoz bevitel, és ezek a műveletek alkalmazásához szükséges feladatok adatainak frissítése. Ez lehet UI teljesítmény javítása érdekében, vagy más, a művelet végrehajtása, ha az esemény bekövetkeztekor figyelői események terjesztését. Például egy bérlistarendszer szolgáltatással való integráció egy költség küldésének webhelyet, hogy az esemény által kiváltott események válaszként a webhelyen végzett fel a webhelyet, mind a bérlistarendszer adatok tárolásához.

- Ha azt szeretné, hogy rugalmasan módosíthatja a materializált modellek és Entitásadatok formátuma, módosíthatja a követelményeket, ha vagy&mdash;CQRS együtt használva&mdash;kell egy olvasási modell, vagy felfedheti azokat a nézeteket.

- CQRS, és végül mind együtt történő használat esetén konzisztencia elfogadható, amíg olvasási modell frissül, vagy a teljesítményre gyakorolt hatást entitásokat és adatainak áttelepítését egy eseményfelhasználó rehidratálása nem elfogadható.

Ez a minta nem lehet a következő esetekben lehet hasznos:

- Kis vagy egyszerű tartományok, rendszerek, amelyeken kevéssé vagy egyáltalán ne üzleti logika vagy útválasztással rendszerek természetes jól használható felügyeleti mechanizmusok hagyományos CRUD adatokat.

- Azok a rendszerek, ahol konzisztencia és a nézetek az adatok a valós idejű frissítések szükségesek.

- Ahol elkérése rendszerek, előzményei és funkciók visszaállítása és visszajátszásos műveletek esetén nincs szükség.

- Rendszerek a mögöttes adatok ütköző frissítések csak nagyon kis előfordulása esetén. Például a rendszerek, hanem frissítése túlnyomórészt hozzáadása.

## <a name="example"></a>Példa

A konferencia rendszer szüksége van egy konferencia befejezett lefoglalások számát úgy, hogy azt is ellenőrzi, hogy vannak-munkaállomásokat marad. Ha egy potenciális résztvevő megpróbál egy foglalási. A rendszer sikerült tárolni a konferencia lefoglalások száma legalább két módon:

- A rendszer egy adatbázisban, amely tartalmazza a foglalási külön egységként sikerült tárolja a lefoglalások teljes számával kapcsolatos információkat. Lefoglalások készített, illetve megszakítva, a rendszer nem növelhető vagy nem ez a szám, megfelelő módon csökkentheti. Ez a megközelítés elméletileg egyszerű, de méretezhetőség hibákat is okozhat, ha nagyszámú résztvevők könyv munkaállomásokat próbál során rövid időn belül. Például az utolsó napja vagy, a foglalási időszakának záró előtt.

- A rendszer sikerült tárolni a lefoglalások és a sikertelen esemény tárolóban tárolt eseményként is rögzíti. Ezek az események visszajátszását által elérhető helyek száma kiszámításához majd azt sikerült. Lehet, hogy ez a megközelítés több méretezhető események immutability miatt. A rendszer csak kell adatokat olvasni az esemény-tároló, vagy adatok hozzáfűzése a esemény tároló. Lefoglalások és a sikertelen esemény információ soha nem módosul.

A következő diagram azt ábrázolja, hogyan lehet, hogy a konferencia felügyeleti rendszer ülések foglalás alrendszere esemény forrás készletével megvalósított.

![Az esemény rögzítéséhez helyfoglalás kapcsolatos információkat a konferencia-kezelési rendszerbe forrás használatával](./_images/event-sourcing-bounded-context.png)


A két hely lefoglalása műveletek sorrendjét a következőképpen történik:

1. A felhasználói felület munkaállomásokat foglalása két résztvevők parancsot ad ki. A parancs egy külön parancskezelő kezeli. Egy adat logika, amely a felhasználói felület különválik és parancsként közzétett kérelmek kezeléséért felelős.

2. A konferencia összes lefoglalását kapcsolatos információkat tartalmazó összesítő összeállított a foglalások és a törlések leíró események lekérdezésével. Az összesítés nevezik `SeatAvailability`, és magában foglal egy tartományi modellt, amely kérdez le, és az adatok módosítása aggregált használatos metódusok közzététele.

    > Figyelembe kell venni néhány optimalizálásokat pillanatképek használ (így nem kell lekérdezni, és a teljes listát az beszerzése a jelenlegi állapotában a összesítés események visszajátszásos), és az összesítés a memóriában gyorsítótárazott másolatának megtartásával.

3. A parancskezelő jelennek meg, ha a tartomány modellt a fenntartásokat metódus meghívja.

4. A `SeatAvailability` összesítés volt fenntartva munkaállomásszámot tartalmazó esemény rögzíti. Összesített vonatkozik események, amikor legközelebb a lefoglalását használandó számítási hány munkaállomásokat maradnak.

5. A rendszer hozzáfűzi az új esemény események az esemény-tárolóban.

Ha egy felhasználó megszakítja a helyet, a rendszer egy hasonló folyamatot követi, azzal a különbséggel parancskezelő generál egy eseményt, ülések megszakítása és fűzi azokat hozzá az esemény tároló parancsot ad ki.

A méretezhetőség érdekében további hatókör biztosít, valamint egy esemény tárolót használó is biztosít a teljes előzmények, illetve a napló, a lefoglalások és konferencia a következménye. Az események az esemény tárolóban a pontos rekord. Nincs szükség más módon összesítések megőrizni, mert a rendszer egyszerűen visszajátszásos az események és az állapot visszaállítása bármely időpontra.

> Ebben a példában található további információ található [esemény forrás bevezetéséről](https://msdn.microsoft.com/library/jj591559.aspx).

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- [Parancsot, és lekérdezi a felelősség elkülönítése (CQRS) mintát](cqrs.md). Az írási tároló, amely az állandó információforrás a CQRS megvalósításának gyakran alapú esemény forrás minta megvalósítása. Ismerteti, hogyan elkülönítse a műveleteket, amelyek az alkalmazás adatokat olvasni az operatív adatokat frissítő külön-felületek használatával.

- [A materializált nézet mintát](materialized-view.md). Egy esemény forrás alapján rendszerben használt adattároló nincs általában kiválóan alkalmas a hatékony lekérdezése. Ehelyett egy közös megoldás, az adatok előfeltöltött nézetek létrehozásához, rendszeres időközönként, vagy az adatok változásakor. Bemutatja, hogyan ehhez.

- [Tranzakció mintát Compensating](compensating-transaction.md). A meglévő adatok esemény sourcing tárolóban nem frissül, inkább új bejegyzések kerülnek, térjen át az entitás állapotát az új értékekkel. Fordított módosítását, mert nem lehet az előző módosítás egyszerűen fordított kell használni kompenzációs bejegyzéseket. Ismerteti, hogyan vonható vissza egy korábbi művelet által végzett munka.

- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). Ha esemény forrás egy külön olvassa el a tároló vagy materializált nézetek, a beolvasott adat nem lesz azonnal konzisztens, ehelyett lesz csak idővel konzisztenssé. A konzisztencia fenntartása körülvevő elosztott adatokon keresztül problémákat foglalja össze.

- [Útmutatás particionálás adatok](https://msdn.microsoft.com/library/dn589795.aspx). Adatok gyakran particionálása használatakor esemény forrás méretezhetőség javítása, csökkentse a versengés és teljesítményének optimalizálásához. Adatok felosztása diszkrét partíció, és az esetlegesen felmerülő problémákat ismerteti.

- Greg Young post [miért érdemes használni a esemény forrás?](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/).
