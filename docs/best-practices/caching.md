---
title: Gyorsítótárazási útmutató
titleSuffix: Best practices for cloud applications
description: Útmutatás a gyorsítótárazáshoz a teljesítmény és a méretezhetőség javítása érdekében.
author: dragon119
ms.date: 05/24/2017
ms.custom: seodec18
ms.openlocfilehash: b7f720b9e08b0316f9967a19e1b93069aa04e55f
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307469"
---
# <a name="caching"></a>Gyorsítótárazás

A gyorsítótárazás egy gyakran alkalmazott módszer, amelynek célja egy rendszer teljesítményének és méretezhetőségének javítása. Ehhez a rendszer ideiglenesen átmásolja a gyakran használt adatokat egy, az alkalmazás közelében lévő gyors tárolóba. Ha ez a gyors adattároló közelebb van az alkalmazáshoz, mint az eredeti forrás, akkor a gyorsabb adatkiszolgálás révén a gyorsítótárazással lényegesen javítható az ügyfélalkalmazások válaszideje.

A gyorsítótárazás akkor a leghatékonyabb, ha egy ügyfélpéldány rendszeresen ugyanazokat az adatokat olvassa be, és különösen akkor, ha az eredeti adattárat az alábbi tulajdonságok jellemzik:

- Viszonylag statikus marad.
- A gyorsítótár sebességéhez képest lassú.
- Magas szintű versengésnek van kitéve.
- Távoli, és a hálózati késleltetés lassú hozzáférést eredményezhet.

## <a name="caching-in-distributed-applications"></a>Gyorsítótárazás az elosztott alkalmazásokban

Az elosztott alkalmazások általában az alábbi stratégiák egyikét vagy mindegyikét alkalmazzák az adatok gyorsítótárazása során:

- Privát gyorsítótár használata, amikor az adatok helyileg vannak tárolva az alkalmazás- vagy szolgáltatáspéldányt futtató számítógépen.
- Megosztott gyorsítótár használata, amely több folyamat és/vagy számítógép által elérhető, közös adatforrásként szolgál.

A gyorsítótárazásra mindkét esetben sor kerülhet az ügyféloldalon és/vagy a kiszolgálóoldalon. Az ügyféloldali gyorsítótárazást az a folyamat végzi el, amely egy rendszer felhasználói felületét biztosítja, például egy webböngészőt vagy egy asztali alkalmazást. A kiszolgálóoldali gyorsítótárazás a távolról futtatott üzleti szolgáltatásokat biztosító folyamattal történik.

### <a name="private-caching"></a>Privát gyorsítótárazás

A legegyszerűbb gyorsítótárazási módszer a memóriában történő tárolás. Az adatok tárolása egyetlen folyamat címtartományában történik, és a tár közvetlenül, az adott folyamatban futó kóddal érhető el. Ez a gyorsítótártípus nagyon gyors hozzáférést biztosít. Igen hatékony módszernek bizonyult közepes mennyiségű statikus adat tárolására, mivel a gyorsítótár méretét jellemzően a folyamatot futtató számítógép szabad memóriájának mennyisége korlátozza.

Ha több adatot kell gyorsítótárazni, mint amennyit a memória fizikailag lehetségessé tesz, a gyorsítótárazott adatokat a helyi fájlrendszerbe is kiírhatja. Ezek hozzáférése lassabb, mint a memóriában tárolt adatoké, de még mindig gyorsabb és megbízhatóbb, mint az adatok hálózaton keresztüli lekérése.

Ha egy, a modellt használó alkalmazásból egyszerre több példányt is futtat, akkor minden alkalmazáspéldány saját, független gyorsítótárral rendelkezik, benne az adatok másolati példányaival.

A gyorsítótárat az eredeti adatok múltbeli pillanatfelvételeként képzelje el. Ha ezek az adatok nem statikusak, akkor valószínű, hogy a különböző alkalmazáspéldányok az adatok különböző verzióit tárolják a gyorsítótárukban. Ezért az ilyen példányok által végrehajtott azonos lekérdezés különböző eredményeket adhat, ahogy az 1. ábrán is látható.

![Memórián belüli gyorsítótár használata egy alkalmazás különböző példányaiban](./images/caching/Figure1.png)

*1. ábra: Egy alkalmazás különböző példányaiban memórián belüli gyorsítótár használata.*

### <a name="shared-caching"></a>Megosztott gyorsítótár használata

A megosztott gyorsítótárral kiküszöbölhető az a memórián belüli gyorsítótár használatakor előforduló probléma, hogy az egyes gyorsítótárakban lévő adatok eltérhetnek egymástól. A megosztott gyorsítótárazással biztosítható, hogy a különböző alkalmazáspéldányok a gyorsítótárazott adatok azonos nézetét lássák. Ehhez a rendszer a gyorsítótár számára egy külön helyet hoz létre, amelyet rendszerint egy különálló szolgáltatás részeként üzemeltet, a 2. ábrán látható módon.

![Megosztott gyorsítótár használata](./images/caching/Figure2.png)

*2. ábra: Megosztott gyorsítótár használata.*

A megosztott gyorsítótár használatának egyik fontos előnye az általa biztosított méretezhetőség. Sok olyan megosztott gyorsítótárazási szolgáltatás van, amely kiszolgálófürtön alapul, és olyan szoftvereket használ, amelyek átlátható módon osztják el az adatokat a fürtön keresztül. Az alkalmazáspéldány egyszerűen elküld egy kérelmet a gyorsítótár-szolgáltatásnak. Az alapul szolgáló infrastruktúra felelős a gyorsítótárazott adatok fürtbeli helyének meghatározásáért. A gyorsítótár további kiszolgálók hozzáadásával egyszerűen méretezhető.

A megosztott gyorsítótár használatának két fő hátránya van:

- Lassabb a hozzáférés, mivel a gyorsítótár tárolása már nem helyileg, az egyes alkalmazáspéldányokban történik.
- Az, hogy külön gyorsítótár-szolgáltatást kell kiépíteni, összetettebbé teheti az alkalmazott megoldást.

## <a name="considerations-for-using-caching"></a>A gyorsítótár használatának szempontjai

A következő szakaszokban részletesebben is ismertetjük a gyorsítótár tervezése és használata során megfontolandó szempontokat.

### <a name="decide-when-to-cache-data"></a>Hogyan lehet megállapítani, hogy mikor van szükség az adatok gyorsítótárazására?

A gyorsítótárazással jelentősen javítható a teljesítmény, a méretezhetőség és a rendelkezésre állás. Minél több adattal dolgozik, és minél nagyobb azon felhasználók száma, akik számára az adatokhoz hozzáférést kell biztosítani, annál inkább jelentkeznek a gyorsítótár használatának előnyei. Ez azért van így, mert a gyorsítótárazás révén csökken az a késleltetés és versengés, amely az eredeti adattárban kialakult a túl sok egyidejű kérelem kezelése miatt.

Előfordulhat például, hogy egy adatbázis csak korlátozott számú párhuzamos kapcsolatot támogat. Ha azonban az adatokat a megosztott gyorsítótárból, nem pedig az alapul szolgáló adatbázisból kérik le, akkor az ügyfélalkalmazás abban az esetben is hozzáfér az adatokhoz, ha éppen nem áll rendelkezésre szabad kapcsolat. Ha pedig az adatbázis nem érhető el, a gyorsítótárban tárolt adatok felhasználásával az ügyfélalkalmazások továbbra is működőképesek maradhatnak.

Érdemes megfontolni a gyorsítótárazás alkalmazását a gyakran beolvasott, de ritkán módosított adatok esetében (ha például bizonyos adatok olvasási műveleteinek száma nagyobb, mint az írási műveleteké). Nem javasoljuk azonban, hogy a gyorsítótárat a kritikus fontosságú információk mérvadó tárolójaként használja. Ehelyett gondoskodjon arról, hogy egy állandó adattárba kerüljön az összes olyan módosítás, amely nem veszhet el az alkalmazásból. Ez azt eredményezi, hogy ha a gyorsítótár nem érhető el, az adattár használatával az alkalmazás továbbra is működőképes maradhat, és nem vesznek el fontos információk.

### <a name="determine-how-to-cache-data-effectively"></a>Az adatok hatékony gyorsítótárazási módszerének meghatározása

A gyorsítótár hatékony használatának kulcsa, hogy meg tudjuk határozni a gyorsítótárazásra leginkább alkalmas adatokat és időpontokat. Az adatok például hozzáadhatók a gyorsítótárhoz akkor, amikor egy alkalmazás először lekéri őket. Ez azt jelenti, hogy az alkalmazásnak csak egyszer kell beolvasnia az adatokat az adattárból, ezt követően a hozzáférés a gyorsítótár használatával már megoldható.

Alternatív megoldásként a gyorsítótár részlegesen vagy teljesen előre is feltölthető az adatokkal, jellemzően az alkalmazás indításakor (ez a megoldás áttöltésként is ismert). Nem feltétlenül ajánlott azonban az áttöltés használata nagy méretű gyorsítótárak esetében, mert ez a megoldás az alkalmazás indításakor hirtelenül és túlzott mértékben leterhelheti az eredeti adattárat.

Gyakran a használati minták elemzése segíthet eldönteni, hogy érdemes-e teljesen vagy részlegesen előre feltölteni a gyorsítótárat, és hogy mely adatokat érdemes e célból kiválasztani. Hasznosnak bizonyulhat például a gyorsítótár feltöltése olyan ügyfelek statikus felhasználóiprofil-adataival, akik rendszeresen (esetleg minden nap) használják az alkalmazást, azokéval azonban nem, akik csak hetente egyszer veszik használatba.

A gyorsítótárazás általában jól működik olyan adatokkal, amelyek nem módosíthatók vagy csak ritkán változnak. Ilyenek lehetnek a referenciainformációk, például egy elektronikus kereskedelmi alkalmazás termék- és díjszabási adatai, illetve olyan megosztott statikus erőforrások, amelyeknek a létrehozása nagyobb költséggel jár. Ezek az adatok részben vagy teljes egészében betölthetők a gyorsítótárba az alkalmazás indításakor, így minimalizálható az erőforrásigény, és fokozható a teljesítmény. Érdemes lehet alkalmazni egy olyan háttérfolyamat is, amely a naprakész állapot biztosítása érdekében rendszeresen frissíti a referenciaadatokat a gyorsítótárban, vagy amely a referenciaadatok módosításakor frissíti a gyorsítótárat.

A gyorsítótárazás kevésbé hasznos, ha dinamikus adatokról van szó, bár vannak kivételek (további információkért lásd lejjebb a „Nagymértékben dinamikus adatok gyorsítótárazása” szakaszt). Ha az eredeti adatok rendszeresen változnak, a gyorsítótárban lévő információk nagyon gyorsan elavulhatnak, illetve a gyorsítótár és az eredeti adattár szinkronizálásának szükséglete csökkenti a gyorsítótárazás hatékonyságát.

Vegye figyelembe, hogy a gyorsítótárnak nem kell tartalmaznia egy-egy entitás minden adatát. Ha például egy adatelem többértékű objektumot jelöl (például egy banki ügyfelet névvel, címmel és számlaegyenleggel), akkor ezen elemek közül egyesek statikusak (például a név és a cím), míg mások (például a számlaegyenleg) dinamikusabban változók lehetnek. Ilyen esetekben hasznos lehet az adatok statikus részét gyorsítótárba helyezni, és csak akkor lekérdezni (vagy kiszámítani) a többi adatot, amikor szükség van rájuk.

Javasoljuk, hogy végezzen teljesítménytesztelést és használatelemzést annak meghatározására, hogy a gyorsítótár előre történő vagy igény szerinti feltöltése, esetleg ezek valamilyen kombinációja jelenti-e a megfelelő megoldást az Ön számára. A döntésnek az adatok változékonyságán és használati módján kell alapulnia. A gyorsítótár-kihasználtság és a teljesítmény elemzése különösen fontos azoknál az alkalmazásoknál, amelyeknek nagy terheléssel kell megbirkózniuk, és hatékonyan méretezhetőnek kell lenniük. Hatékonyan méretezhető alkalmazások esetén hasznosnak bizonyulhat például a gyorsítótár áttöltése, hogy csökkenjen az adattárat csúcsidőben érő terhelés.

A gyorsítótárazással az is elkerülhető, ismétlődő számításokat kelljen elvégezni, miközben az alkalmazás fut. Ha egy művelet adatátalakítást vagy bonyolult számításokat végez, akkor a művelet eredményei elmenthetők a gyorsítótárba. Ha később ugyanazt a számítást kell elvégezni, az alkalmazás egyszerűen lekérheti az eredményeket a gyorsítótárból.

Az alkalmazások módosíthatják a gyorsítótárban tárolt adatokat. Azt javasoljuk azonban, hogy a gyorsítótárat olyan átmeneti adattárként kezelje, amelynek az adatai bármikor elveszhetnek. Értékes adatokat ne tároljon kizárólag a gyorsítótárban; győződjön meg róla, hogy az adatok továbbra is megtalálhatók az eredeti adattárban. Így minimálisra csökken az adatvesztés esélye akkor, ha a gyorsítótár nem érhető el.

### <a name="cache-highly-dynamic-data"></a>Nagymértékben dinamikus adatok gyorsítótárazása

Ha gyorsan változó információkat egy állandó adattárolóban tárol, az többletterhelést okozhat a rendszerben. Vegyünk például egy olyan eszközt, amely folyamatos jelentést küld az állapotról vagy egyéb mérőszámokról. Ha egy alkalmazás nem tárolja ezeket az adatokat a gyorsítótárban, mivel a gyorsítótárazott adatok így szinte mindig elavultak lennének, akkor ugyanez az információknak az adattárban való tárolására és onnan történő lekérésére is igaz lehet. Az adatok mentéséhez és beolvasásához szükséges idő alatt az adatok már módosulhattak is.

Ilyen esetekben érdemes megfontolni, hogy milyen előnyökkel járhat a dinamikus információk tárolása közvetlenül a gyorsítótárban, az állandó adattár helyett. Ha az adatok nem kritikus fontosságúak és nem kell őket naplózni, akkor nem számít, ha egy-egy módosítás elvész.

### <a name="manage-data-expiration-in-a-cache"></a>A gyorsítótárazott adatok lejáratának kezelése

A legtöbb esetben a gyorsítótárban tárolt adatok az eredeti adattárban lévő adatok másolatai. A gyorsítótárba helyezést követően az eredeti adattárban lévő adatok módosulhatnak, ami a gyorsítótárazott adatok elavulását eredményezi. Számos gyorsítótárazási rendszer teszi lehetővé a gyorsítótár konfigurálását arra, hogy elévültnek jelölje meg az adatokat, így csökkenthető az az időszak, amíg elavult adatokat tárol.

Az elévült adatokat a rendszer eltávolítja a gyorsítótárból, és az alkalmazásnak az eredeti adattárból kell visszaállítania az adatokat (ezek az újonnan lekért adatok aztán visszahelyezhetők a gyorsítótárba). A gyorsítótár konfigurálásakor beállíthat egy alapértelmezett elévülési szabályzatot. Több gyorsítótár-szolgáltatásban az egyes objektumok lejárati idejét is meg lehet szabni, ha azokat programozott módon tárolja a gyorsítótárban. Egyes gyorsítótárak lehetővé teszik a lejárati idő abszolút vagy mozgó értékként történő megadását, amely alapján a rendszer eltávolítja az adott elemeket a gyorsítótárból, ha a megszabott idő alatt nem próbáltak meg hozzájuk férni. Ez a beállítás felülbírál bármely, a teljes gyorsítótárra érvényes elévülési szabályzatot, de csak a megadott objektumokra vonatkozik.

> [!NOTE]
> Körültekintően vegye figyelembe a gyorsítótár és az abban lévő objektumok lejárati idejét. Ha az túl rövid, az objektumok túl hamar lejárnak, és így kevésbé érvényesülnek a gyorsítótár használatának előnyei. Ha viszont az időszakot túl hosszúra állítja be, azzal az adatok elévülését kockáztatja.

Amennyiben az adatokat hosszabb ideig nem távolítja el, előfordulhat, hogy a gyorsítótár megtelik. Ebben az esetben, ha új elemeket próbál hozzáadni a gyorsítótárhoz, sor kerülhet bizonyos elemek kényszerített eltávolítására – ezt a folyamatot kiürítésnek nevezzük. A gyorsítótár-szolgáltatások jellemzően a legrégebben használt (least-recently-used, LRU) adatokat ürítik ki, de általában lehetőség van a szabályzat felülírására és az ilyen elemek kiürítésének megakadályozására. Ez azonban a gyorsítótárban rendelkezésre álló memória megtelésének kockázatával jár, és az elemet a gyorsítótárhoz hozzáadni próbáló alkalmazás futtatása egy kivétellel meghiúsulhat.

Egyes gyorsítótárazási megoldások további kiürítési szabályzatokat is biztosíthatnak. Többféle típusú kiürítési szabályzat létezik. Ezek a következők:

- Legutóbbi használaton alapuló szabályzat (ha az adatokra a továbbiakban feltehetően nem lesz szükség).
- Elsőnek be, elsőnek ki szabályzat (először mindig a legrégebbi adatok kiürítésére kerül sor).
- Kiváltott eseményen (például módosított adatokon) alapuló explicit eltávolítási szabályzat.

### <a name="invalidate-data-in-a-client-side-cache"></a>Adatok érvénytelenítése egy ügyféloldali gyorsítótárban

Az ügyféloldali gyorsítótárban tárolt adatokat általában nem tekintjük ahhoz a szolgáltatáshoz tartozónak, amely adatokat szolgáltat az ügyfél számára. A szolgáltatások közvetlenül nem kényszeríthetik az ügyfeleket az ügyféloldali gyorsítótár információinak hozzáadására vagy eltávolítására.

Ezért előfordulhat, hogy a nem megfelelően konfigurált gyorsítótárat használó ügyfelek elavult információkkal fognak dolgozni. Ha például a gyorsítótár elévülési szabályzatainak megvalósítása nem megfelelő, akkor lehetséges, hogy az ügyfelek helyileg gyorsítótárazott, elavult információkat fognak használni, miközben az eredeti adatforrás adatai már módosultak.

Ha olyan webalkalmazást épít ki, amely HTTP-kapcsolaton keresztül szolgáltat adatokat, akkor implicit módon kényszeríthet egy webes ügyfelet (például böngészőt vagy webproxyt) a legfrissebb információk lekérésére. Ezt akkor teheti meg, ha egy erőforrás az erőforrás URI-jának módosításával frissül. A webes ügyfelek általában az erőforrás URI-ját használják az ügyféloldali gyorsítótár kulcsaként, tehát ha az URI módosul, akkor a webes ügyfél figyelmen kívül hagyja az adott erőforrás előzőleg gyorsítótárazott verzióit, és helyette az új verziót olvassa be.

## <a name="managing-concurrency-in-a-cache"></a>Az egyidejűség kezelése a gyorsítótárakban

A gyorsítótárak tervezése során gyakran az a cél, hogy egy alkalmazás több példánya által közösen használhatók legyenek. Mindegyik alkalmazáspéldány képes olvasni és módosítani a gyorsítótár adatait. Következésképpen a megosztott adattárral kapcsolatban felmerülő egyidejűségi problémák a gyorsítótáraknál is fellépnek. Olyan helyzetben, amikor egy alkalmazásnak módosítania kell a gyorsítótárban tárolt adatokat, előfordulhat, hogy gondoskodni kell arról, hogy az alkalmazás egyik példánya által elvégzett frissítések ne írhassák felül a másik példány által végrehajtott módosításokat.

Az adatok természetétől és az ütközések valószínűségétől függően kétféle egyidejűségi megközelítés létezik:

- **Az optimista**. Közvetlenül az adatok frissítése előtt az alkalmazás ellenőrzi, hogy a gyorsítótár adatai módosítva lettek-e a lekérésük óta. Ha az adatok nem változtak, akkor a módosítás végrehajtható. Ellenkező esetben az alkalmazásnak kell meghatároznia, hogy elvégzi-e a frissítést. (A döntés alapjául szolgáló üzleti logika az adott alkalmazásra jellemző lesz.) Ez a megközelítés olyan esetekben alkalmazható, ahol a frissítések ritkák, vagy ahol az ütközések előfordulása nem valószínű.
- **A pesszimista**. Az adatok lekérésekor az alkalmazás zárolja az adatokat a gyorsítótárban, nehogy egy másik példány módosíthassa őket. Ez a folyamat megakadályozza ugyan az ütközéseket, de blokkolhat olyan egyéb példányokat is, amelyeknek ugyanazokat az adatokat kell feldolgozniuk. A pesszimista egyidejűségi megközelítés befolyásolhatja a megoldás méretezhetőségét, és alkalmazása csak rövid ideig tartó műveletek esetén javasolt. Akkor lehet ideális megoldás, ha az ütközések nagyobb valószínűséggel fordulnak elő, különösen, ha egy alkalmazás több elemet is frissít a gyorsítótárban, és gondoskodni kell arról, hogy a módosítások következetesek legyenek.

### <a name="implement-high-availability-and-scalability-and-improve-performance"></a>Magas rendelkezésre állás és méretezhetőség megvalósítása, illetve a teljesítmény javítása

Kerülje a gyorsítótár elsődleges adattárként való használatát; erre az eredeti adattár szolgál, amelyből a rendszer feltölti a gyorsítótárat. Az eredeti adattár feladata az adatok megőrzése.

Ügyeljen arra, hogy a használt megoldásokban ne kapcsolódjanak kritikus függőségek a megosztott gyorsítótár-szolgáltatások elérhetőségéhez. Az alkalmazásoknak akkor is működőképesnek kell maradniuk, ha a megosztott gyorsítótárat biztosító szolgáltatás nem érhető el. Addig sem szabad lefagyniuk vagy meghiúsulniuk, amíg a gyorsítótár-szolgáltatás újraindul.

Ezért az alkalmazásoknak képesnek kell lenniük a gyorsítótár-szolgáltatás rendelkezésre állási állapotának észlelésére, illetve az eredeti adattárra való visszaváltásra, ha a gyorsítótár nem érhető el. Ebben az esetben az [áramköri megszakítási minta](../patterns/circuit-breaker.md) használható eredményesen. A gyorsítótárat üzemeltető szolgáltatás visszaállítható, és amint elérhetővé válik, a gyorsítótár újból feltölthető, mivel az adatokat a rendszer az eredeti adattárból olvassa be, egy olyan stratégiát követve, amilyen például a [gyorsítótár-feltöltési minta](../patterns/cache-aside.md).

Ha azonban az alkalmazás visszavált az eredeti adattárra, amikor a gyorsítótár ideiglenesen nem érhető el, az hatással lehet a rendszer méretezhetőségére. A gyorsítótár helyreállítása közben az eredeti adattárat eláraszthatják az adatkérések, ami időtúllépéseket és sikertelen kapcsolatokat eredményezhet.

Érdemes lehet létrehozni egy helyi, privát gyorsítótárat mindegyik alkalmazáspéldányhoz a megosztott gyorsítótár mellett, amelyhez az összes alkalmazáspéldány hozzá tud férni. Amikor az alkalmazás lekér egy adott elemet, először a helyi, majd a megosztott gyorsítótárban, végül pedig az eredeti adattárban kereshet. A helyi gyorsítótár a megosztott gyorsítótárban vagy (ha a megosztott gyorsítótár nem érhető el) az adatbázisban lévő adatokból is feltölthető.

Ennél a megközelítésnél körültekintő konfigurálásra van szükség annak elkerülésére, hogy a helyi gyorsítótár túlságosan elavulttá váljon a megosztott gyorsítótárhoz képest, de a helyi gyorsítótár megfelelő pufferként működhet, ha a megosztott gyorsítótár nem érhető el. Ez a struktúra látható a 3. ábrán.

![Megosztott gyorsítótár egy helyi, privát gyorsítótár használata](./images/caching/Caching3.png)

*3. ábra: Egy helyi, privát gyorsítótár használata egy megosztott gyorsítótárban.*

A viszonylag hosszú élettartamú adatokat tartalmazó, nagy méretű gyorsítótárak támogatása érdekében egyes gyorsítótár-szolgáltatások biztosítják a magas rendelkezésre állás lehetőségét, amely automatikus feladatátvételt hajt végre, ha a gyorsítótár nem érhető el. Ez a megközelítés általában magában foglalja az elsődleges gyorsítótár-kiszolgálón tárolt gyorsítótárazott adatok másodlagos gyorsítótár-kiszolgálóra történő replikációját, és ha az elsődleges kiszolgáló meghibásodik, vagy ha a kapcsolat megszakad, akkor a rendszer a másodlagos kiszolgáló használatára vált.

A több célhelyre való írás okozta késleltetés csökkentése érdekében a másodlagos kiszolgálóra történő replikáció aszinkron módon is megvalósítható. Ilyenkor a rendszer az adatokat az elsődleges kiszolgáló gyorsítótárába írja. Ez a megközelítés annak lehetőségét, hogy meghibásodás esetén egyes gyorsítótárazott adatok elveszhetnek, de az időarány, amíg az adatok kis méretűnek kell lenniük a gyorsítótár teljes mérete képest vezet.

Ha a megosztott gyorsítótár mérete nagy, akkor előnyös lehet a gyorsítótárban tárolt adatok csomópontokra történő particionálása a versengés kialakulási esélyének csökkentése és a méretezhetőség javítása érdekében. Sok megosztott gyorsítótár támogatja a csomópontok dinamikus hozzáadását (és eltávolítását), valamint az adatok újraegyensúlyozását a partíciók között. Ez a megközelítés magában foglalhatja a fürtözést is, amikor a csomópontgyűjtemény az ügyfélalkalmazások számára egyetlen osztatlan gyorsítótárként jelenik meg. A rendszeren belül azonban az adatok egy előre megadott elosztási stratégia szerint vannak szétosztva a csomópontok között, így egyenlítve ki a terhelést. További információ a lehetséges particionálási módszerekről: [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx).

A fürtözéssel a gyorsítótár rendelkezésre állása is megnövelhető. Ha egy csomópont meghibásodik, a gyorsítótár fennmaradó része továbbra is elérhető marad. A fürtözést gyakran használják a replikációval és a feladatátvétellel együtt. Az egyes csomópontok replikálhatók, és a replika a csomópont meghibásodásakor gyorsan online állapotba helyezhető.

Az olvasási és írási műveletek többsége nagy valószínűséggel egyetlen adatértéket vagy objektumot tartalmaz. Esetenként azonban szükség lehet nagy adatmennyiségek gyors tárolására vagy lekérdezésére. A gyorsítótár-áttöltés például több száz vagy ezer elem gyorsítótárba írásával is járhat. Előfordulhat, hogy egy alkalmazásnak egy adott kérelem részeként nagy mennyiségű kapcsolódó elemet is le kell kérnie a gyorsítótárból.

Számos nagy méretű gyorsítótár biztosít kötegműveleteket erre a célra. Az ilyen műveletekkel az ügyfélalkalmazások nagy mennyiségű tételt csomagolhatnak össze egyetlen kérelemmé, így csökkenthető a sok kisebb kérelem végrehajtásával járó többletterhelés.

## <a name="caching-and-eventual-consistency"></a>Gyorsítótárazás és végleges konzisztencia

Ahhoz, hogy a gyorsítótár-feltöltési minta működni tudjon, a gyorsítótárat feltöltő alkalmazáspéldánynak hozzáféréssel kell rendelkeznie az adatok legfrissebb és konzisztens verzióihoz. A végleges konzisztenciát megvalósító rendszerekben (például egy replikált adattárban) ez nem feltétlenül biztosítható.

Egy alkalmazás egyik példánya módosíthat egy adatelemet, amely érvénytelenítheti az adott elem gyorsítótárazott verzióját. Az alkalmazás egy másik példánya megpróbálhatja ezt az elemet kiolvasni egy gyorsítótárból, ami nem fog sikerülni, így az adatokat az adattárból olvassa ki, és hozzáadja a gyorsítótárhoz. Ha azonban az adattár nem lett teljes mértékben szinkronizálva a többi replikával, akkor megtörténhet, hogy az alkalmazáspéldány a régi értéket olvassa ki, és azt tölti fel a gyorsítótárba.

Az adatkonzisztencia kezelésére vonatkozó további információkért lásd az [adatkonzisztencia ismertetését](https://msdn.microsoft.com/library/dn589800.aspx).

### <a name="protect-cached-data"></a>A gyorsítótárazott adatok védelme

A használt gyorsítótár-szolgáltatástól függetlenül mindig át kell gondolni, hogyan védheti meg a gyorsítótárban tárolt adatokat az illetéktelen hozzáféréstől. A következő két fő területtel kell foglalkoznunk:

- A gyorsítótárban lévő adatok adatvédelme.
- Adatvédelem a gyorsítótár és a gyorsítótárat használó alkalmazás közötti adatátvitel során.

A gyorsítótárban lévő adatok védelme érdekében a gyorsítótár-szolgáltatás olyan hitelesítési mechanizmust alkalmazhat, amely a következők meghatározását követeli meg az alkalmazásoktól:

- Azon identitások, amelyek hozzáférhetnek a gyorsítótárban lévő adatokhoz.
- Azon (olvasási és írási) műveletek, amelyeket az identitások jogosultak elvégezni.

Az adatok olvasásához és írásához kapcsolódó terhelés csökkentése érdekében az identitás a gyorsítótárban lévő összes adatot felhasználhatja, miután írási és/vagy olvasási hozzáférést kapott a gyorsítótárhoz.

Ha korlátozni kell a gyorsítótárazott adatok részhalmazainak hozzáférését, a következők szerint járhat el:

- Ossza fel a gyorsítótárat partíciókra (különböző gyorsítótár-kiszolgálók használatával), és csak azon partíciókhoz biztosítson hozzáférést, amelyeket az identitások használhatnak.
- Más-más kulcsok használatával titkosítsa az egyes részhalmazok adatait, és a titkosítási kulcsokat csak azoknak az identitásoknak adja meg, amelyeknek mindegyik részhalmazhoz hozzá kell férniük. Az ügyfélalkalmazások ugyan továbbra is képesek lehetnek lekérni a gyorsítótár bármely adatát, de csak azokat tudják visszafejteni, amelyek titkosítási kulcsaival rendelkeznek.

Biztosítani kell ezen kívül a gyorsítótár bejövő és kimenő adatainak védelmét is. Ehhez a hálózati infrastruktúra által biztosított azon biztonsági szolgáltatásokra fog támaszkodni, amelyeket az ügyfélalkalmazások a gyorsítótárhoz történő kapcsolódáshoz használnak. Ha a gyorsítótár megvalósítása az ügyfélalkalmazásokat üzemeltető az ugyanazon szervezeten belüli helyszíni kiszolgáló használata, majd maga a hálózat elkülönítése nem feltétlenül igényelnek, hogy további lépéseket. Ha a távoli gyorsítótár nyilvános hálózaton (például az interneten) történő használatához TCP- vagy HTTP-kapcsolat szükséges, fontolja meg az SSL alkalmazását.

## <a name="considerations-for-implementing-caching-in-azure"></a>Az Azure gyorsítótár megvalósítási szempontjai

Az [Azure Redis Cache](/azure/redis-cache/) egy nyílt forráskódú Redis-gyorsítótár, amely szolgáltatásként fut egy Azure-adatközpontban. Olyan gyorsítótárazási szolgáltatást biztosít, amely bármely Azure-alkalmazásból elérhető, függetlenül attól, hogy az alkalmazás felhőszolgáltatásként, webhelyként vagy egy Azure-beli virtuális gépen belül lett létrehozva. A gyorsítótáron olyan ügyfélalkalmazások osztozhatnak, amelyek rendelkeznek a megfelelő hozzáférési kulccsal.

Az Azure Redis Cache egy nagy teljesítményű gyorsítótárazási megoldás, amely rendelkezésre állási, méretezhetőségi és biztonsági szolgáltatásokat nyújt. Általában egy vagy több dedikált gépen elosztott szolgáltatásként fut. A gyors hozzáférés biztosításához megpróbálja a lehető legtöbb információt eltárolni a memóriában. Az architektúrának az a célja, hogy kevesebb lassú I/O műveletet kelljen végrehajtani, így alacsonyabb legyen a késleltetés, és nagyobb a teljesítmény.

Az Azure Redis Cache az ügyfélalkalmazások által használt sokféle API-val kompatibilis. Ha már rendelkezik olyan alkalmazásokkal, amelyek helyileg futtatják az Azure Redis Cache-t, akkor az Azure Redis Cache gyors migrálási útvonalat biztosít a felhőben történő gyorsítótárazáshoz.

### <a name="features-of-redis"></a>A Redis jellemzői

A Redis több, mint egy egyszerű gyorsítótár-kiszolgáló. Átfogó parancskészlettel ellátott, elosztott, memóriabeli adatbázisról van szó, amely számos gyakori alkalmazási helyzetet támogat. Ezekről az ebben a dokumentumban az a szakasz Redis-gyorsítótárazás használata című szakaszát. Ez a szakasz a Redis néhány fő funkcióját foglalja össze.

### <a name="redis-as-an-in-memory-database"></a>A Redis használata memóriabeli adatbázisként

A Redis az olvasási és az írási műveleteket is támogatja. Redis írási vagy egy helyi pillanatképfájlba vagy egy csak hozzáfűzéssel bővíthető naplófájlba rendszeres időközönként tárolja a rendszer hibája biztosítható. Ez nem a helyzet a legtöbb gyorsítótárban (amelyeket kell adattárolóknak kell tekinteni).

Az összes írási művelet aszinkron módon lesz elvégezve, és nem akadályozzák az ügyfeleket az adatok írásában vagy olvasásában. Amikor a Redis elindul, a pillanatkép- vagy naplófájlból olvassa ki az adatokat, és azokat használja a memórián belüli gyorsítótár létrehozásához. További információkat a Redis webhelyén elérhető, a [Redis adatmegőrzésével](https://redis.io/topics/persistence) foglalkozó témakörben talál.

> [!NOTE]
> A Redis nem garantálja, hogy egy katasztrofális hiba esetén az összes írási műveletet elmenti, de a legrosszabb esetben is csak néhány másodpercnyi adatmennyiséget veszíthet el. Ne felejtse el, hogy a gyorsítótár nem kezelendő mérvadó adatforrásként, és a gyorsítótárat használó alkalmazások felelősek azért, hogy a kritikus fontosságú adatokat sikeresen elmentsék a megfelelő adattárba. További információkért lásd: a [gyorsítótár-feltöltési minta](../patterns/cache-aside.md).

#### <a name="redis-data-types"></a>Redis-adattípusok

A Redis egy olyan kulcs-érték tároló, ahol az értékek egyszerű típusokat vagy összetett adatstruktúrákat (például kivonatokat, listákat és készleteket) tartalmazhatnak. A Redis többféle atomi művelet elvégzését teszi lehetővé ezeken az adattípusokon. A kulcsok lehetnek állandóak vagy korlátozott időtartamúak, amelynek lejártakor a kulcs és annak értéke automatikusan el lesz távolítva a gyorsítótárból. A Redis-kulcsokról és -értékekről további információkat a Redis webhelyén elérhető, [a Redis adattípusait és absztrakt entitásait bemutató](https://redis.io/topics/data-types-intro) témakörben talál.

#### <a name="redis-replication-and-clustering"></a>Redis-replikáció és -fürtözés

A Redis támogatja a fölé-/alárendelt típusú replikációt a rendelkezésre állás biztosítása és az átviteli sebesség szinten tartása érdekében. A Redis-főcsomópontokra irányuló írási műveletek egy vagy több alárendelt csomópontra replikálhatók. Az olvasási műveleteket a főcsomópont és az alárendelt csomópontok is kiszolgálhatják.

Ha hálózatszakadás következik be, az alárendelt csomópontok továbbra is szolgáltathatnak adatokat, majd a kapcsolat helyreállásakor a rendszer transzparens módon újraszinkronizálja azokat a főcsomóponttal. További részleteket a Redis webhelyén elérhető, a [replikációval](https://redis.io/topics/replication) foglalkozó oldalon talál.

A redis fürtözésre is lehetőséget biztosít, amely lehetővé teszi, hogy transzparens módon adatok particionálása történő particionálását a kiszolgálók között, és a terhelés elosztását. Ez a funkció javítja a méretezhetőséget, mivel a gyorsítótár méretének növekedésével új Redis-kiszolgálókat lehet hozzáadni, az adatok pedig újraparticionálhatók.

A fürt mindegyik kiszolgálója replikálható fölé- vagy alárendelt típusú replikációval. Ez mindegyik fürtcsomópont rendelkezésre állását biztosítja. A fürtözésről és a horizontális skálázásról további információkat a Redis webhelyén elérhető, a [Redis-fürt oktatóanyagát](https://redis.io/topics/cluster-tutorial) tartalmazó oldalon talál.

### <a name="redis-memory-use"></a>A Redis-memória használata

A Redis-gyorsítótár mérete véges, és a gazdagépen rendelkezésre álló erőforrásoktól függ. A Redis-kiszolgálók konfigurálásakor megadhatja az általuk maximálisan felhasználható memória mennyiségét. Egy lejárati idővel ellátott kulcsot is a beállíthat a Redis-gyorsítótárban, amely a lejáratát követően automatikusan eltávolítódik a gyorsítótárból. Ezzel a funkcióval megakadályozható, hogy a memórián belüli gyorsítótár régi vagy elavult adatokkal legyen feltöltve.

Ahogy a memória megtelik, a Redis adott szabályzatok alapján automatikusan ki tudja üríteni a kulcsokat és azok értékeit. Az alapértelmezett szabályzat az LRU (least recently used, legrégebben használt), de más szabályzatokat is kiválaszthat, így például a kulcsok véletlenszerű kiürítését, vagy a kiürítés kikapcsolását (ekkor nem lehet új elemeket hozzáadni a megtelt gyorsítótárhoz). További információkat [a Redis LRU-gyorsítótárként történő használatával](https://redis.io/topics/lru-cache) foglalkozó témakörben talál.

### <a name="redis-transactions-and-batches"></a>Redis-tranzakciók és -kötegek

A Redis lehetővé teszi az ügyfélalkalmazások számára, hogy olyan műveletsort küldhessenek el, amelyek atomi tranzakciókként adatokat olvasnak és írnak a gyorsítótárba. A tranzakció összes parancsa garantáltan egymás után fog lefutni, a többi párhuzamos ügyfél által kiadott parancsok beékelődése nélkül.

Ezek azonban nem valódi tranzakciók, hiszen a végrehajtásukért egy relációs adatbázis felel. A tranzakciók feldolgozása két szakaszból áll: az első a parancsok üzenetsorba állítása, a második pedig a parancsok futtatása. A parancsok sorkezelési szakaszában az ügyfél beküldi azokat a parancsokat, amelyekből a tranzakció áll. Ha ekkor valamilyen hiba történik (például szintaktikai hiba lép fel vagy helytelen számú paraméter van megadva), akkor a Redis megtagadja a teljes tranzakció feldolgozását, és elveti azt.

A futtatási szakaszban a Redis egymás után végrehajtja a sorba állított parancsokat. Ha ebben a fázisban egy parancsot nem lehet végrehajtani, a Redis a sorban következő parancsra vált, és nem állítja vissza a már futtatott parancsok eredményét. Ezzel az egyszerűsített tranzakciós módszerrel szinten tartható a teljesítmény, és elkerülhetők a versengés okozta teljesítményproblémák.

A Redis egyfajta optimista zárolást valósít meg a konzisztencia megőrzése érdekében. A Redis használatával történő tranzakció-végrehajtásról és zárolásról további információkat a Redis webhelyén elérhető, a [tranzakciókat ismertető oldal](https://redis.io/topics/transactions) tartalmaz.

A Redis a kérelmek nem tranzakciós célú kötegelését is támogatja. Az a Redis-protokoll, amellyel az ügyfelek parancsokat küldenek egy Redis-kiszolgálóra, lehetővé teszi az ügyfelek számára, hogy ugyanazon kérelem részeként egy egész műveletsort is elküldhessenek. Ezzel csökkenthető a hálózat csomagtöredezettségének mértéke. A köteg feldolgozásakor a rendszer minden parancsot végrehajt. Ha a parancsok bármelyike helytelenül van megformázva, akkor az(ok)at a rendszer elutasítja (ami egy tranzakció esetén nem történik meg), a többi parancsot azonban végrehajtja. A kötegben lévő parancsok feldolgozási sorrendje nem garantálható.

### <a name="redis-security"></a>A Redis biztonsági funkciói

A Redis kizárólagos célja az adatokhoz való gyors hozzáférés biztosítása, és csak megbízható ügyfelek számára elérhető megbízható környezetben futtatható. A Redis támogatja a jelszavas hitelesítésen alapuló, korlátozott biztonságot nyújtó modelleket. (A hitelesítés teljesen el is hagyható, de ezt nem javasoljuk.)

Az összes hitelesített ügyfél ugyanazt a globális jelszót használja, és ugyanazokhoz az erőforrásokhoz fér hozzá. Ha ennél átfogóbb bejelentkezési biztonsági megoldásra van szüksége, akkor létre kell hoznia egy saját biztonsági réteget a Redis-kiszolgáló előtt, és minden ügyfélkérelemnek ezen kell áthaladnia. Meg kell akadályozni, hogy a Redishez közvetlenül hozzáférhessenek nem megbízható vagy nem hitelesített ügyfelek.

A parancsok elérését a letiltásukkal vagy az átnevezésükkel korlátozhatja (és az új neveket csak az arra jogosult ügyfelek kapják meg).

A Redis közvetlenül nem támogatja az adattitkosítás semmilyen formáját, így a teljes kódolást az ügyfélalkalmazásoknak kell elvégezniük. Ezenkívül a Redis nem biztosít semmiféle átviteli biztonsági megoldást sem. Ha gondoskodnia kell az adatok védelméről a hálózati átvitel során, erre a célra SSL-proxy alkalmazását javasoljuk.

További információkat a Redis webhelyén elérhető, [a Redis biztonsági funkcióival foglalkozó](https://redis.io/topics/security) oldalon talál.

> [!NOTE]
> Az Azure Redis Cache saját biztonsági réteget biztosít az ügyfelek csatlakozásához. A Redis háttérkiszolgálói a nyilvános hálózaton nem érhetőek el.

### <a name="azure-redis-cache"></a>Azure Redis Cache

Az Azure Redis Cache az Azure-adatközpontokon üzemeltetett Redis-kiszolgálókhoz biztosít hozzáférést. Az Azure Redis Cache hozzáférés-vezérlési és biztonsági előtérrendszerként funkcionál. Telepíthet egy gyorsítótár az Azure portal használatával.

A portálon számos előre meghatározott konfiguráció érhető el. Ilyen konfigurációra példa egy dedikált szolgáltatásként működő, 53 GB-os gyorsítótár, amely támogatja az SSL-kommunikációt (adatvédelmi célból) és a fölé/alárendelt típusú replikációt 99,9%-os rendelkezésre állást garantáló SLA-val, de a konfiguráció lehet akár egy replikáció nélküli, megosztott hardveren futó, 250 MB-os gyorsítótár is (garantált rendelkezésre állás nélkül).

Az Azure Portalon a gyorsítótár kiürítési szabályzatát is beállíthatja, és vezérelheti a gyorsítótárhoz való hozzáférést a felhasználók adott szerepkörökhöz történő hozzáadásával. Ezek a tagok által elvégezhető műveleteket meghatározó szerepkörök a következők: tulajdonos, közreműködő és olvasó. A Tulajdonos szerepkör tagjai például teljes körű, a gyorsítótárra (beleértve a biztonsági megoldásokat) és annak tartalmára vonatkozó szabályozási jogosultsággal bírnak, a Közreműködő szerepkör tagjai olvasási és írási műveleteket végezhetnek a gyorsítótárban, az Olvasó szerepkör tagjai pedig csak adatokat kérhetnek le a gyorsítótárból.

Az adminisztratív feladatok többségét az Azure Portalon kell végrehajtani. Emiatt a Redis standard verziójában elérhető adminisztratív parancsok közül több nem áll rendelkezésre, így például a konfiguráció szoftveres módosításának, a Redis-kiszolgáló leállításának, további alárendelt elemek konfigurálásának vagy az adatok lemezre történő kényszerített mentésének lehetősége.

Az Azure Portal kényelmesen használható grafikus felülete lehetővé teszi a gyorsítótár teljesítményének monitorozását. Megtekinthető például a létrehozott kapcsolatok, a végrehajtott kérelmek, az olvasási és írási műveletek száma, valamint a gyorsítótár-találatok és -tévesztések aránya. Ezen információk alapján meghatározhatja a gyorsítótár hatékonyságát, és szükség esetén átválthat más konfigurációra, vagy módosíthatja a kiürítési szabályzatot.

Olyan riasztásokat is létrehozhat, amelyek e-mail-üzeneteket küldenek egy rendszergazdának, ha egy vagy több kritikus jelentőségű mérőszám értéke kívül esik a várható tartományon. Érdemes lehet riasztani a rendszergazdát például akkor, ha az elmúlt órában a gyorsítótár-tévesztések száma meghaladott egy adott értéket, mert ez azt jelentheti, hogy a gyorsítótár túl kicsi, vagy az adatok kiürítésére túl gyorsan kerül sor.

A gyorsítótár CPU-, memória- és hálózathasználata is monitorozható.

Az Azure Redis Cache létrehozásával és konfigurálásával kapcsolatos további információkat és példákat az Azure blog [Azure Redis Cache-t körbejáró](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) oldalán talál.

## <a name="caching-session-state-and-html-output"></a>A gyorsítótárazási munkamenet állapota és HTML-kimenete

Azure-beli webes szerepkörök használatával futtatott ASP.NET-webalkalmazások létrehozásakor a munkamenet állapotinformációit és a HTML-kimenetet egy Azure Redis Cache-ben mentheti el. Az Azure Redis Cache munkamenetállapot-szolgáltatója engedélyezi a munkamenet-információk megosztását az ASP.NET-webalkalmazás különböző példányai között. Ez nagyon hasznos olyan webfarm-forgatókönyvek esetében, ahol az ügyfél-kiszolgáló affinitás nem áll rendelkezésre, és a munkamenetadatok memóriában történő gyorsítótárazása nem lenne megfelelő megoldás.

Az Azure Redis Cache munkamenetállapot-szolgáltatójának használata számos előnnyel jár, például:

- A munkamenet-állapot ASP.NET-webalkalmazások sok példányával osztható meg.
- Jobb a méretezhetőség.
- Szabályozott, párhuzamos hozzáférést lehet biztosítani ugyanazon munkamenet-állapotadatokhoz több olvasó és egyetlen író számára.
- Tömörítéssel menthető a memória és javítható a hálózati teljesítmény.

További információkért lásd [az Azure Redis Cache ASP.NET munkamenetállapot-szolgáltatójával](/azure/redis-cache/cache-aspnet-session-state-provider/) foglalkozó cikket.

> [!NOTE]
> Ne használja az Azure Redis Cache munkamenetállapot-szolgáltatóját Azure-környezeten kívül futtatott ASP.NET-alkalmazásokkal. Az Azure-on kívüli gyorsítótár hozzáférésének késleltetése miatt előfordulhat, hogy az adatok gyorsítótárazásának teljesítménybeli előnyei nem használhatók ki.

Hasonlóképpen, az Azure Redis Cache kimeneti gyorsítótár-szolgáltatója az ASP.NET-webalkalmazások által létrehozott HTTP-válaszok mentését is lehetővé teszi. Az Azure Redis Cache kimeneti gyorsítótár-szolgáltatójának használatával csökkenthető az összetett HTML-kimeneteket renderelő alkalmazások válaszideje. A hasonló válaszokat visszaadó alkalmazáspéldányok a gyorsítótárban lévő megosztott kimeneti töredékeket használhatják a HTML-kimenet újbóli létrehozása helyett. További információkért lásd [az Azure Redis Cache ASP.NET-es kimeneti gyorsítótár-szolgáltatóját](/azure/redis-cache/cache-aspnet-output-cache-provider/) ismertető cikket.

## <a name="building-a-custom-redis-cache"></a>Egyéni Redis-gyorsítótár kiépítése

Az Azure Redis Cache a háttérbeli Redis-kiszolgálók előtérrendszereként funkcionál. Ha olyan speciális konfigurációra van szükség, amelyet az Azure Redis Cache nem támogat (például 53 GB-nál nagyobb méretű gyorsítótárra), az Azure-beli virtuális gépek segítségével saját Redis-kiszolgálókat is létrehozhat és üzemeltethet.

Ez a folyamat meglehetősen összetett lehet, hiszen ha replikációt szeretne végrehajtani, akkor esetenként több virtuális gépet is létre kell hoznia, amelyek a fő- és alárendelt csomópontok szerepét töltik be. Továbbá, ha fürtöt kíván létrehozni, akkor több fő- és alárendelt kiszolgálóra lesz szüksége. Egy magas szintű rendelkezésre állást és méretezhetőséget biztosító, minimális méretű fürtözött replikációs topológia legalább hat virtuális gépből, azaz három pár fő- és alárendelt kiszolgálóból áll (egy fürtben legalább három főcsomópontnak kell lennie).

A késleltetés minimalizálása érdekében a fölé/alárendelt pároknak egymáshoz közel kell lenniük. Ha azonban a gyorsítótárban lévő adatokat az őket legtöbbször használó alkalmazásokhoz közel kívánja elhelyezni, az egyes párok futtathatók különböző régiókban található Azure-adatközpontokban is. Példa Azure-beli virtuális gépként futtatott Redis-csomópont kiépítésére és konfigurálására: [A Redis futtatása CentOS Linux rendszerű virtuális gépen az Azure-ban](https://blogs.msdn.microsoft.com/tconte/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure/) (blogbejegyzés).

> [!NOTE]
> Vegye figyelembe, hogy ha egyéni Redis-gyorsítótárat hoz létre, akkor Ön felel a szolgáltatás monitorozásáért, felügyeletéért és biztonságáért.

## <a name="partitioning-a-redis-cache"></a>A Redis-gyorsítótár particionálása

A gyorsítótár particionálása a gyorsítótár több számítógépre történő felosztását jelenti. Ez a struktúra számos előnnyel jár egyetlen gyorsítótár-kiszolgáló használatával szemben, amelyek például a következők lehetnek:

- Sokkal nagyobb gyorsítótár hozható létre, mint amelyet egyetlen kiszolgálón tárolni lehetne.
- Az adatok eloszthatók a kiszolgálók között, így javul a rendelkezésre állás. Ha egy kiszolgáló meghibásodik vagy elérhetetlenné válik, az általa tárolt adatok sem érhetők el, a többi kiszolgálón található adatok viszont továbbra is hozzáférhetők maradnak. Gyorsítótárak esetén ez nem jelent nagy problémát, mert a gyorsítótárazott adatok csak az adatbázisban tárolt adatok átmeneti másolati példányai. A hozzáférhetetlenné váló kiszolgáló gyorsítótárában tárolt adatok egy másik kiszolgálóra gyorsítótárába is áthelyezhetők.
- A terhelés elosztható a kiszolgálók között, ezáltal javul a teljesítmény és méretezhetőség.
- Az adatok a hozzájuk férő felhasználókhoz földrajzilag közel helyezhetők el, így csökken a késleltetés mértéke.

Gyorsítótárak esetében a particionálás leggyakrabban használt módszere a horizontális skálázás. Ilyenkor az egyes partíciók (vagy szilánkok) önmagukban is egy-egy Redis-gyorsítótárként működnek. Az adatokat horizontális skálázási logika alapján irányítják egy megadott partícióra, amely logika többféle megközelítést is alkalmazhat az adatok elosztására. További információkat a horizontális skálázás megvalósításáról a [horizontális skálázási mintát](../patterns/sharding.md) ismertető cikkben talál.

A Redis-gyorsítótár particionálásának végrehajtása során a következő megközelítéseket alkalmazhatja:

- *Kiszolgálóoldali lekérdezés továbbítása.* Az ügyfélalkalmazás kérelmet küld a gyorsítótár részét képező valamelyik Redis-kiszolgálónak (nagy valószínűséggel a legközelebbinek). Mindegyik Redis-kiszolgáló tárolja a rajta található partíciót leíró metaadatokat, és arra vonatkozóan is tartalmaz információkat, hogy mely partíciók találhatók más kiszolgálókon. A Redis-kiszolgáló megvizsgálja az ügyfélkérelmet. Ha az helyileg megoldható, akkor elvégzi a kért műveletet. Ha nem oldható meg, akkor továbbítja a kérést a megfelelő kiszolgálóra. Erről a Redis-fürtözéssel megvalósított modellről további részleteket a Redis webhelyén elérhető, a [Redis-fürt oktatóanyagát](https://redis.io/topics/cluster-tutorial) tartalmazó oldalon talál. A Redis-fürtözés átlátható az ügyfélalkalmazások számára, és a fürthöz további Redis-kiszolgálók is hozzáadhatók (és az adatok újraparticionálhatók) anélkül, hogy az ügyfeleket újra kellene konfigurálni.
- *Ügyféloldali particionálás.* Ebben a modellben az ügyfélalkalmazás (valószínűleg kódtár formájában elérhető) logikája irányítja a megfelelő Redis-kiszolgálókra a kérelmeket. Ez a módszer az Azure Redis Cache esetében is alkalmazható. Hozzon létre több Azure Redis Cache-t (mindegyik adatpartícióhoz egyet-egyet), és léptesse életbe az ügyféloldali logikát, amely a kérelmeket a megfelelő gyorsítótárba irányítja. Ha a particionálási séma módosul (például további Azure Redis Cache-ek létrehozása esetén), akkor az ügyfélalkalmazások újrakonfigurálására lehet szükség.
- *Proxyval támogatott particionálás.* Ebben a sémában az ügyfélalkalmazások egy közvetítőként szolgáló proxyszolgáltatásnak küldenek kérelmeket, amely értelmezi az adatok particionálását, majd a kérelmet a megfelelő Redis-kiszolgálóra irányítja. Ez a módszer is alkalmazható az Azure Redis Cache esetében; a proxyszolgáltatás Azure-felhőszolgáltatásként is megvalósítható. Így összetettebb feladattá válik a szolgáltatás megvalósítása, és hosszabb időre lehet szükség a kérelmek végrehajtásához, mint az ügyféloldali particionálás esetében.

A Redis használatával történő particionálásról további információkat a Redis webhelyén elérhető [Particionálás: adatok felosztása több Redis-példány között](https://redis.io/topics/partitioning) oldalon talál.

### <a name="implement-redis-cache-client-applications"></a>Redis gyorsítótár-ügyfélalkalmazások megvalósítása

A Redis számos, különböző programozási nyelveken írt ügyfélalkalmazást támogat. Ha a .NET-keretrendszer használatával hoz létre új alkalmazásokat, ehhez a StackExchange.Redis ügyfélkódtár használatát javasoljuk. Ez a kódtár egy olyan .NET-keretrendszerbeli objektummodellt biztosít, amely kivonatolja a Redis-kiszolgálóhoz való csatlakozás, a parancsküldés és a válaszfogadás részleteit. A Visual Studióban NuGet-csomagként érhető el. Ugyanezzel a kódtárral kapcsolódhat egy Azure Redis-gyorsítótárhoz vagy egy virtuális gépen lévő egyéni Redis-gyorsítótárhoz.

A Redis-kiszolgálóhoz való csatlakozáshoz használja a `ConnectionMultiplexer` osztály statikus `Connect` metódusát. A metódus által létrehozott kapcsolat úgy van kialakítva, hogy az ügyfélalkalmazás teljes élettartama alatt használható legyen, és ugyanezt a kapcsolatot több egyidejű szál is használhassa. Redis-műveletek végrehajtásakor ne hajtson végre minden alkalommal újracsatlakozást és leválasztást, mert ez ronthatja a teljesítményt.

Megadhatja a kapcsolati paramétereket, például a Redis-gazdagép címét és a jelszót. Azure Redis Cache használatakor a jelszó nem vagy az elsődleges vagy másodlagos kulcsot, amely az Azure Redis Cache jön létre az Azure felügyeleti portál használatával.

Miután csatlakozott a Redis-kiszolgálóhoz, beszerezhet egy leírót a gyorsítótárként szolgáló Redis-adatbázishoz. A Redis-kapcsolat ehhez a `GetDatabase` metódust biztosítja. Ezután a `StringGet` és a `StringSet` metódussal kérdezhet le elemeket a gyorsítótárból és menthet el benne adatokat. Ezek a metódusok paraméterként egy kulcsot várnak, majd visszaadják az egyező értékű elemet a gyorsítótárból (`StringGet`), vagy ezzel a kulccsal adják hozzá az elemet a gyorsítótárhoz (`StringSet`).

A Redis-kiszolgáló helyétől függően számos műveletnél számíthatunk késleltetésre, amíg a kérelem eljut a kiszolgálóhoz, illetve a válasz az ügyfélhez. A StackExchange kódtár számos metódus aszinkron verzióját biztosítja, amelyek segítségével szinten tartható az ügyfélalkalmazások válaszkészsége. Ezek a metódusok támogatják a [feladatalapú aszinkron minta](/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) a .NET-keretrendszer.

A következő kódrészlet a `RetrieveItem` nevű metódust mutatja be. A példa a Redis és a StackExchange-kódtáron alapuló gyorsítótár-feltöltési minta megvalósítását szemlélteti. A metódus egy sztring formátumú kulcsérték használatával megkísérli lekérni a megfelelő elemet a Redis-gyorsítótárból a `StringGetAsync` metódus (a `StringGet` aszinkron verziója) meghívásával.

Ha az adott elem nem található, akkor a rendszer az alapul szolgáló adatforrásból kéri le a `GetItemFromDataSourceAsync` metódussal (helyi metódus, nem része a StackExchange-kódtárnak). A rendszer ezután a `StringSetAsync` metódussal hozzáadja az elemet a gyorsítótárhoz, így legközelebb gyorsabban le lehet majd kérdezni.

```csharp
// Connect to the Azure Redis cache
ConfigurationOptions config = new ConfigurationOptions();
config.EndPoints.Add("<your DNS name>.redis.cache.windows.net");
config.Password = "<Redis cache key from management portal>";
ConnectionMultiplexer redisHostConnection = ConnectionMultiplexer.Connect(config);
IDatabase cache = redisHostConnection.GetDatabase();
...
private async Task<string> RetrieveItem(string itemKey)
{
    // Attempt to retrieve the item from the Redis cache
    string itemValue = await cache.StringGetAsync(itemKey);

    // If the value returned is null, the item was not found in the cache
    // So retrieve the item from the data source and add it to the cache
    if (itemValue == null)
    {
        itemValue = await GetItemFromDataSourceAsync(itemKey);
        await cache.StringSetAsync(itemKey, itemValue);
    }

    // Return the item
    return itemValue;
}
```

A `StringGet` és a `StringSet` metódus nem korlátozódik a lekérésre és a sztringértékek tárolására. Ezek a metódusok minden, bájttömbként szerializált elemet támogatnak. Ha egy a .NET-objektumot kell elmentenie, bájtstreamként szerializálhatja, majd a `StringSet` metódussal a gyorsítótárba írhatja.

Hasonlóképpen, a `StringGet` metódus használatával beolvashatja az adott objektumot a gyorsítótárból, majd .NET-objektumként deszerializálhatja azt. Az alábbi kódban az IDatabase-illesztő bővítési metódusai (egy Redis-kapcsolat `GetDatabase` metódusa egy `IDatabase` objektumot ad vissza), illetve néhány olyan mintakód látható, amelyek ezeket a metódusokat használják egy `BlogPost` objektum gyorsítótárbeli olvasásához és írásához:

```csharp
public static class RedisCacheExtensions
{
    public static async Task<T> GetAsync<T>(this IDatabase cache, string key)
    {
        return Deserialize<T>(await cache.StringGetAsync(key));
    }

    public static async Task<object> GetAsync(this IDatabase cache, string key)
    {
        return Deserialize<object>(await cache.StringGetAsync(key));
    }

    public static async Task SetAsync(this IDatabase cache, string key, object value)
    {
        await cache.StringSetAsync(key, Serialize(value));
    }

    static byte[] Serialize(object o)
    {
        byte[] objectDataAsStream = null;

        if (o != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream())
            {
                binaryFormatter.Serialize(memoryStream, o);
                objectDataAsStream = memoryStream.ToArray();
            }
        }

        return objectDataAsStream;
    }

    static T Deserialize<T>(byte[] stream)
    {
        T result = default(T);

        if (stream != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream(stream))
            {
                result = (T)binaryFormatter.Deserialize(memoryStream);
            }
        }

        return result;
    }
}
```

Az alábbi kód egy `RetrieveBlogPost` nevű metódust szemléltet, amely ezeket a bővítő metódusokat használja egy szerializálható `BlogPost` objektum gyorsítótárbeli olvasásához és írásához, a gyorsítótár-feltöltési minta alapján:

```csharp
// The BlogPost type
[Serializable]
public class BlogPost
{
    private HashSet<string> tags;

    public BlogPost(int id, string title, int score, IEnumerable<string> tags)
    {
        this.Id = id;
        this.Title = title;
        this.Score = score;
        this.tags = new HashSet<string>(tags);
    }

    public int Id { get; set; }
    public string Title { get; set; }
    public int Score { get; set; }
    public ICollection<string> Tags => this.tags;
}
...
private async Task<BlogPost> RetrieveBlogPost(string blogPostKey)
{
    BlogPost blogPost = await cache.GetAsync<BlogPost>(blogPostKey);
    if (blogPost == null)
    {
        blogPost = await GetBlogPostFromDataSourceAsync(blogPostKey);
        await cache.SetAsync(blogPostKey, blogPost);
    }

    return blogPost;
}
```

A Redis támogatja a parancsok adatcsatornás feldolgozását, ha egy ügyfélalkalmazás több aszinkron kérelmet küld. A parancsok szigorú sorrendben történő fogadása és megválaszolása helyett a Redis képes az ugyanazt a kapcsolatot használó kérelmek multiplexálására.

Ez a módszer a hatékonyabb hálózathasználat révén hozzájárul a késleltetés csökkentéséhez. Az alábbi kódrészlet két ügyfél adatainak egyidejű lekérdezésére mutat példát. A kód két kérelmet küld el, majd végrehajt egy másik műveletet (amely itt nem látható), miközben az eredmények beérkezésére vár. A gyorsítótár-objektum `Wait` metódusa hasonló a .NET-keretrendszer `Task.Wait` metódusához:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
var task1 = cache.StringGetAsync("customer:1");
var task2 = cache.StringGetAsync("customer:2");
...
var customer1 = cache.Wait(task1);
var customer2 = cache.Wait(task2);
```

További információkat az Azure Redis Cache-sel kompatibilis ügyfélalkalmazások írásáról az [Azure Redis Cache dokumentációjában](https://azure.microsoft.com/documentation/services/cache/) talál. A [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md) is tartalmaz további tudnivalókat.

Az ugyanezen a webhelyen található, [folyamatokkal és multiplexerekkel](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) foglalkozó oldalon további információkat talál az aszinkron műveletekről és az adatcsatornás feldolgozásról a Redis és a StackExchange kódtár használatával. 

## <a name="using-redis-caching"></a>A Redis-gyorsítótárazás használata

A gyorsítótárazás során a Redis legegyszerűbben kulcs-érték párok formájában használható, ahol az érték egy tetszőleges hosszúságú, nem értelmezett sztring, amely bármilyen bináris adatot tartalmazhat. (Tulajdonképpen bájttömb egy karakterláncként kezelhető szó). Ezt a forgatókönyvet a jelen cikk korábbi, „Redis gyorsítótár-ügyfélalkalmazások megvalósítása” című szakaszában mutattuk be.

Vegye figyelembe, hogy a kulcsok nem értelmezett adatokat is tartalmaznak, így bármilyen bináris adat használható kulcsként. Minél hosszabb azonban a kulcs, a tárolás során annál több helyet foglal, és annál hosszabb ideig tart a keresési műveletek végrehajtása. A használhatóság és az egyszerű karbantartás érdekében gondosan tervezze meg a kulcsterületet, és használjon jelentéssel bíró (de ne túl részletes) kulcsokat.

Használjon strukturált kulcsokat, amilyen például az „ügyfél:100”, hogy az ügyfélkulcsot a 100-as azonosítóval, ne pedig csak a „100” értékkel jelölje. Ez a séma lehetővé teszi, hogy egyszerűen megkülönböztethessük a különböző adattípusokat tároló értékeket. Használhatja például a „rendelések:100” kulcsot is a 100-as azonosítójú megrendelés kulcsának megjelölésére.

Az egydimenziós bináris sztringekon kívül a Redis kulcs-érték párok értéke strukturáltabb információkat is tartalmazhat, például listákat, (rendezett vagy nem rendezett) készleteket és kivonatokat. A Redis olyan átfogó parancskészletet biztosít, amely segítségével ezek a típusok módosíthatók, és a parancsok közül a legtöbb elérhető a .NET-keretrendszert használó alkalmazások számára egy ügyfélkódtáron (például a StackExchange-en) keresztül. A típusokról és a módosításukhoz használható parancsokról részletesebb áttekintést a Redis webhelyén elérhető, [a Redis adattípusait és absztrakt entitásait bemutató](https://redis.io/topics/data-types-intro) oldalon talál.

Ez a szakasz ezen adattípusok és parancsok néhány gyakori felhasználási esetét foglalja össze.

### <a name="perform-atomic-and-batch-operations"></a>Atomi és kötegműveletek végrehajtása

A Redis számos „lekérés és beállítás” típusú atomi műveletet támogat, amelyeket sztringértékeken lehet végrehajtani. E műveletek segítségével kiküszöbölhetők azok a versengésből származó esetleges veszélyek, amelyek a külön `GET` és `SET` parancsok használatakor előfordulhatnak. A rendelkezésre álló műveletek a következők:

- `INCR`, `INCRBY`, `DECR` és `DECRBY` – ezek atomi növelési és csökkentési műveletek, amelyeket egész számú numerikus adatértékeken lehet végrehajtani. A StackExchange-kódtár biztosítja az `IDatabase.StringIncrementAsync` és `IDatabase.StringDecrementAsync` metódusok túlterhelt verzióit a fenti műveletek elvégzéséhez, és a gyorsítótárban tárolt eredményt adja vissza. A következő kódrészlet ezen metódusok használatát mutatja be:

  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  await cache.StringSetAsync("data:counter", 99);
  ...
  long oldValue = await cache.StringIncrementAsync("data:counter");
  // Increment by 1 (the default)
  // oldValue should be 100
  
  long newValue = await cache.StringDecrementAsync("data:counter", 50);
  // Decrement by 50
  // newValue should be 50
  ```

- `GETSET` – lekéri a kulcshoz társított értéket, és egy új értékre módosítja. A StackExchange-kódtár az `IDatabase.StringGetSetAsync` metóduson keresztül teszi elérhetővé ezt a műveletet. Az alábbi kódrészlet erre a metódusra mutat példát. Ez a kód az előző példából származó „data:counter” (adat:számláló) kulcs aktuális értékét adja vissza. Ezt követően nulla értékre állítja vissza a kulcs értékét, mindezt egyazon művelet részeként:

  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  string oldValue = await cache.StringGetSetAsync("data:counter", 0);
  ```

- `MGET` és `MSET` – egyetlen műveletként képes visszaadni vagy módosítani sztringértékek egy készletét. Az `IDatabase.StringGetAsync` és `IDatabase.StringSetAsync` metódusok túlterheltek ennek a funkciónak a támogatásához, ahogy a következő példában is látható:

  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  // Create a list of key-value pairs
  var keysAndValues =
      new List<KeyValuePair<RedisKey, RedisValue>>()
      {
          new KeyValuePair<RedisKey, RedisValue>("data:key1", "value1"),
          new KeyValuePair<RedisKey, RedisValue>("data:key99", "value2"),
          new KeyValuePair<RedisKey, RedisValue>("data:key322", "value3")
      };
  
  // Store the list of key-value pairs in the cache
  cache.StringSet(keysAndValues.ToArray());
  ...
  // Find all values that match a list of keys
  RedisKey[] keys = { "data:key1", "data:key99", "data:key322"};
  // values should contain { "value1", "value2", "value3" }
  RedisValue[] values = cache.StringGet(keys);

  ```

Több műveletet is egyesíthet egyetlen Redis-tranzakcióba, ahogy azt a jelen cikk korábbi, „Redis-tranzakciók és -kötegek” című részében ismertettük. A StackExchange-kódtár az `ITransaction` illesztőn keresztül biztosít támogatást a tranzakciók számára.

Az `ITransaction` objektumok létrehozása az `IDatabase.CreateTransaction` metódussal történik. Az `ITransaction` objektum által biztosított metódusokkal hívhat meg parancsokat a tranzakcióhoz.

Az `ITransaction` illesztő metódusok olyan készletéhez biztosít hozzáférést, amely hasonló az `IDatabase` illesztőn keresztül elérhetőhöz, de ezek mindegyike aszinkron metódus. Ez azt jelenti, hogy a végrehajtásukra csak az `ITransaction.Execute` metódus meghívásakor kerül sor. Az `ITransaction.Execute` metódus által visszaadott érték azt jelzi, hogy a tranzakció sikeresen létrejött-e (igaz) vagy sem (hamis).

Az alábbi kódrészlet két számláló ugyanazon tranzakció részeként történő növelésére és csökkentésére mutat példát:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
ITransaction transaction = cache.CreateTransaction();
var tx1 = transaction.StringIncrementAsync("data:counter1");
var tx2 = transaction.StringDecrementAsync("data:counter2");
bool result = transaction.Execute();
Console.WriteLine("Transaction {0}", result ? "succeeded" : "failed");
Console.WriteLine("Result of increment: {0}", tx1.Result);
Console.WriteLine("Result of decrement: {0}", tx2.Result);
```

Ne feledje, hogy a Redis-tranzakciók eltérnek a relációs adatbázisok tranzakcióitól. Az `Execute` metódus egyszerűen sorba állítja a futtatni kívánt tranzakció összes parancsát, és ha ezek közül bármelyik hibás formátumú, akkor leállítja a tranzakciót. Ha minden parancs sikeresen sorba lett állítva, akkor a futtatásuk aszinkron módon történik.

Ha egy parancs végrehajtása meghiúsul, a többi parancs feldolgozása tovább folytatódik. Ha ellenőriznie kell, hogy egy parancs sikeresen befejeződött-e, akkor a kapcsolódó művelet **Result** (Eredmény) tulajdonsága segítségével kell lekérnie a parancs eredményét, ahogy az a fenti példában látható. A **Result** tulajdonság olvasásakor a rendszer a feladat bejezéséig blokkolja a hívó szálat.

További információkért lásd: [Redis-tranzakciókkal](https://stackexchange.github.io/StackExchange.Redis/Transactions).

Kötegműveletek végrehajtásához a StackExchange-kódtár `IBatch` illesztőjét használhatja. Ez az illesztő metódusok olyan készletéhez biztosít hozzáférést, amely hasonló az `IDatabase` illesztőn keresztül elérhetőhöz, de ezek mindegyike aszinkron metódus.

Az `IBatch` objektumok létrehozása az `IDatabase.CreateBatch` metódussal történik, majd az `IBatch.Execute` metódussal futtathatja a köteget, ahogy az alábbi példában látható. Ez a kód egyszerűen beállít egy sztringértéket, növeli és csökkenti az előző példában is használt számlálókat, és megjeleníti az eredményeket:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
IBatch batch = cache.CreateBatch();
batch.StringSetAsync("data:key1", 11);
var t1 = batch.StringIncrementAsync("data:counter1");
var t2 = batch.StringDecrementAsync("data:counter2");
batch.Execute();
Console.WriteLine("{0}", t1.Result);
Console.WriteLine("{0}", t2.Result);
```

Ne feledje: ha egy kötegbeli parancs hibás formátum miatt meghiúsul, a többi parancs futtatása – a tranzakcióktól eltérően – továbbra is lehetséges marad. Az `IBatch.Execute` metódus nem jelzi, hogy a végrehajtás sikeres vagy sikertelen-e.

### <a name="perform-fire-and-forget-cache-operations"></a>Nem követendő gyorsítótár-műveletek végrehajtása

A Redis parancsjelzők használatával támogatja a nem követendő műveleteket. Az ilyen műveleteket az ügyfél kezdeményezi, de nem érdekli az eredmény, és nem várja meg a parancs befejeződését. Az alábbi példa egy INCR-parancs nem követendő műveletként való végrehajtását mutatja be:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
await cache.StringSetAsync("data:key1", 99);
...
cache.StringIncrement("data:key1", flags: CommandFlags.FireAndForget);
```

### <a name="specify-automatically-expiring-keys"></a>Automatikusan lejáró kulcsok megadása

A Redis-gyorsítótárban tárolt elemekhez megadhat egy időtúllépési értéket is, amelynek lejáratát követően a rendszer automatikusan eltávolítja az adott elemet a gyorsítótárból. A `TTL` paranccsal lekérdezheti, hogy mennyi idő van még hátra a kulcs lejártáig. Ezt a parancsot a StackExchange-alkalmazások az `IDatabase.KeyTimeToLive` metódussal érhetik el.

A következő kódrészlet egy 20 másodperces kulcslejárati idő beállítását és a kulcs hátralévő idejének lekérdezését mutatja be:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration time of 20 seconds
await cache.StringSetAsync("data:key1", 99, TimeSpan.FromSeconds(20));
...
// Query how much time a key has left to live
// If the key has already expired, the KeyTimeToLive function returns a null
TimeSpan? expiry = cache.KeyTimeToLive("data:key1");
```

Az EXPIRE parancs használatával a lejárati időt egy meghatározott dátumra és időre is beállíthatja. Ez a parancs a StackExchange-kódtárban érhető el, mint `KeyExpireAsync` metódus:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration date of midnight on 1st January 2015
await cache.StringSetAsync("data:key1", 99);
await cache.KeyExpireAsync("data:key1",
    new DateTime(2015, 1, 1, 0, 0, 0, DateTimeKind.Utc));
...
```

> [!TIP]
> A DEL parancs használatával bármely elemet eltávolíthat a gyorsítótárból. Ez a parancs a StackExchange kódtárban `IDatabase.KeyDeleteAsync` metódusként érhető el.

### <a name="use-tags-to-cross-correlate-cached-items"></a>Címkék használata a gyorsítótárazott elemek összevetéséhez

A Redis-készlet olyan elemek gyűjteménye, amelyek ugyanazt a kulcsot használják. A készletek az SADD paranccsal hozhatók létre. Az adott készletben lévő elemek az SMEMBERS paranccsal kérhetők le. A StackExchange-kódtár az SADD parancsot az `IDatabase.SetAddAsync`, az SMEMBERS parancsot pedig az `IDatabase.SetMembersAsync` metódussal hajtja végre.

A meglévő készleteket új készletek létrehozásához az SDIFF (különbség beállítása), az SINTER (metszet beállítása) és az SUNION (unió beállítása) parancsok használatával egyesítheti. A StackExchange-kódtár ezeket a műveleteket az `IDatabase.SetCombineAsync` metódusban egyesíti. A metódus első paramétere az elvégzendő műveletet határozza meg.

Az alábbi kódrészletek azt mutatják be, hogyan alkalmazhatók a készletek a kapcsolódó elemek gyűjteményének gyors tárolására és lekérdezésére. A kód által használt `BlogPost` típust a jelen cikk korábbi, „Redis gyorsítótár-ügyfélalkalmazások megvalósítása” című szakaszában mutattuk be.

A `BlogPost` objektum a következő négy mezőt tartalmazza: azonosító, cím, rangsorolási pontszám és címkék gyűjteménye. Az alábbi első kódrészlet azokat a mintaadatokat mutatja be, amelyeket a rendszer a `BlogPost` objektumok C# listájának feltöltéséhez használ:

```csharp
List<string[]> tags = new List<string[]>
{
    new[] { "iot","csharp" },
    new[] { "iot","azure","csharp" },
    new[] { "csharp","git","big data" },
    new[] { "iot","git","database" },
    new[] { "database","git" },
    new[] { "csharp","database" },
    new[] { "iot" },
    new[] { "iot","database","git" },
    new[] { "azure","database","big data","git","csharp" },
    new[] { "azure" }
};

List<BlogPost> posts = new List<BlogPost>();
int blogKey = 1;
int numberOfPosts = 20;
Random random = new Random();
for (int i = 0; i < numberOfPosts; i++)
{
    blogKey++;
    posts.Add(new BlogPost(
        blogKey,                  // Blog post ID
        string.Format(CultureInfo.InvariantCulture, "Blog Post #{0}",
            blogKey),             // Blog post title
        random.Next(100, 10000),  // Ranking score
        tags[i % tags.Count]));   // Tags--assigned from a collection
                                  // in the tags list
}
```

Az egyes `BlogPost` objektumokhoz tartozó címkéket tárolhatja készletként egy Redis-gyorsítótárban, és minden egyes készlethez hozzárendelheti a `BlogPost` azonosítóját. Ez teszi lehetővé az alkalmazások számára, hogy gyorsan megtalálják az adott blogbejegyzéshez tartozó összes címkét. Az ellenkező irányban végzett keresés engedélyezéséhez és egy adott címkével ellátott összes blogbejegyzés megkereséséhez létrehozhat egy másik készletet, amely a kulcsban található címkeazonosítóra hivatkozó blogbejegyzéseket tartalmazza:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Tags are easily represented as Redis Sets
foreach (BlogPost post in posts)
{
    string redisKey = string.Format(CultureInfo.InvariantCulture,
        "blog:posts:{0}:tags", post.Id);
    // Add tags to the blog post in Redis
    await cache.SetAddAsync(
        redisKey, post.Tags.Select(s => (RedisValue)s).ToArray());

    // Now do the inverse so we can figure how which blog posts have a given tag
    foreach (var tag in post.Tags)
    {
        await cache.SetAddAsync(string.Format(CultureInfo.InvariantCulture,
            "tag:{0}:blog:posts", tag), post.Id);
    }
}
```

Ezzel a struktúrával sok gyakori lekérdezés nagyon hatékonyan végrehajtható. Megkeresheti és megjelenítheti például az 1. blogbejegyzés összes címkéjét, a következőhöz hasonlóan:

```csharp
// Show the tags for blog post #1
foreach (var value in await cache.SetMembersAsync("blog:posts:1:tags"))
{
    Console.WriteLine(value);
}
```

Az 1. és a 2. blogbejegyzés közös címkéit a következő, metszést beállító művelettel keresheti meg:

```csharp
// Show the tags in common for blog posts #1 and #2
foreach (var value in await cache.SetCombineAsync(SetOperation.Intersect, new RedisKey[]
    { "blog:posts:1:tags", "blog:posts:2:tags" }))
{
    Console.WriteLine(value);
}
```

Így pedig megkereshet minden olyan blogbejegyzést, amely egy adott címkét tartalmaz:

```csharp
// Show the ids of the blog posts that have the tag "iot".
foreach (var value in await cache.SetMembersAsync("tag:iot:blog:posts"))
{
    Console.WriteLine(value);
}
```

### <a name="find-recently-accessed-items"></a>Legutóbb elért elemek megkeresése

Számos alkalmazás esetében szükség van a legutóbb elért elemek megkeresésére. Előfordulhat például, hogy egy blog információkat szeretne megjeleníteni a legutóbb olvasott blogbejegyzésekről.

Ez a feladat egy Redis-lista segítségével hajtható végre. A Redis-lista olyan elemeket tartalmaz, amelyek ugyanazt a kulcsot használják. A lista kétvégű üzenetsorként működik. Az elemeket a lista bármelyik végére leküldheti az LPUSH (leküldés balra) és az RPUSH (leküldés jobbra) paranccsal. Az elemeket a lista bármelyik végéről lekérdezheti az LPOP és az RPOP paranccsal. Elemkészlet is visszaadható az LRANGE és RRANGE parancsokkal.

Az alábbi kódrészletek azt mutatják be, hogyan végezhetők el ezek a műveletek a StackExchange-kódtár használatával. Ez a kód az előző példa `BlogPost` típusát használja. Mivel a blogbejegyzéseket felhasználók olvassák, az `IDatabase.ListLeftPushAsync` metódus a blogbejegyzés címét egy olyan listára küldi le, amely a Redis-gyorsítótárban lévő „blog: recent_posts” kulcshoz van társítva.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:recent_posts";
BlogPost blogPost = ...; // Reference to the blog post that has just been read
await cache.ListLeftPushAsync(
    redisKey, blogPost.Title); // Push the blog post onto the list
```

Ahogy a felhasználók további blogbejegyzéseket olvasnak el, azok címeit is ugyanarra a listára küldi le a rendszer. A lista rendezésére a címek hozzáadási sorrendjében kerül sor. A legutóbb olvasott blogbejegyzések a lista bal oldalán jelennek meg. (Ha ugyanazt a blogbejegyzést többször is olvasták, több bejegyzésben is szerepel a listában.)

A legutóbb olvasott bejegyzések címeit az `IDatabase.ListRange` metódussal jelenítheti meg. Ez a metódus a listát alkotó kulcsot, valamint a kiindulási és a végpontot használja. A következő kód 10 blogbejegyzés címét (0–9. elem) kérdezi le a lista bal oldali végéről:

```csharp
// Show latest ten posts
foreach (string postTitle in await cache.ListRangeAsync(redisKey, 0, 9))
{
    Console.WriteLine(postTitle);
}
```

Vegye figyelembe, hogy a `ListRangeAsync` metódus nem távolít el elemeket a listáról. Erre az `IDatabase.ListLeftPopAsync` és `IDatabase.ListRightPopAsync` metódusokat használhatja.

Ha nem szeretné, hogy a lista folyamatosan egyre hosszabb legyen, rendszeres időközönként kiselejtezhet elemeket a listáról. Az alábbi kódrészlet azt mutatja be, hogyan távolítható el az összes elem a listáról az öt bal szélső elem kivételével:

```csharp
await cache.ListTrimAsync(redisKey, 0, 5);
```

### <a name="implement-a-leader-board"></a>Ranglista létrehozása

Alapértelmezett esetben a készletekben lévő elemek nincsenek meghatározott sorrendbe állítva. A készleteket a ZADD paranccsal (a StackExchange-kódtár `IDatabase.SortedSetAdd` metódusával) rendszerezheti. A tételek rendezése egy pontszámnak nevezett numerikus érték használatával történik, amelyet paraméterként adunk meg a parancs számára.

Az alábbi kódrészlet egy blogbejegyzés címét adja hozzá egy rendezett listához. Ebben a példában mindegyik blogbejegyzéshez tartozik egy-egy pontszám mező is, amely a blogbejegyzés rangsorolását tartalmazza.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:post_rankings";
BlogPost blogPost = ...; // Reference to a blog post that has just been rated
await cache.SortedSetAddAsync(redisKey, blogPost.Title, blogPost.Score);
```

A blogbejegyzések címeit és pontszámait növekvő sorrendben az `IDatabase.SortedSetRangeByRankWithScores` metódussal lehet lekérdezni:

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(redisKey))
{
    Console.WriteLine(post);
}
```

> [!NOTE]
> A StackExchange-kódtár az `IDatabase.SortedSetRangeByRankAsync` metódust is biztosítja, amely az eredményeket pontszám szerinti sorrendben adja vissza, de nem adja vissza a pontszámokat.

Az elemek csökkenő pontszám szerinti sorrendben is lekérdezhetők, illetve korlátozható a visszaadott elemek száma, ha további paramétereket adunk meg az `IDatabase.SortedSetRangeByRankWithScoresAsync` metódusnak. A következő példa a 10 legmagasabb pontszámmal rendelkező blogbejegyzés címét és pontszámát jeleníti meg:

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(
                               redisKey, 0, 9, Order.Descending))
{
    Console.WriteLine(post);
}
```

A következő példa az `IDatabase.SortedSetRangeByScoreWithScoresAsync` metódust használja, amellyel egy adott pontszámtartományra korlátozható a visszaadott elemek köre:

```csharp
// Blog posts with scores between 5000 and 100000
foreach (var post in await cache.SortedSetRangeByScoreWithScoresAsync(
                               redisKey, 5000, 100000))
{
    Console.WriteLine(post);
}
```

### <a name="message-by-using-channels"></a>Üzenetküldés csatornák használatával

Amellett, hogy gyorsítótárként működnek, a Redis-kiszolgálók nagy teljesítményű közzétevői/előfizetői mechanizmuson alapuló üzenetküldésre is használhatók. Az ügyfélalkalmazások előfizethetnek egy csatornára, amelyen más alkalmazások vagy szolgáltatások üzeneteket tudnak közzétenni. Az előfizetéssel rendelkező alkalmazások megkapják ezeket az üzeneteket, és fel tudják őket dolgozni.

A Redis a SUBSCRIBE parancsot biztosítja az ügyfélalkalmazások számára a csatornákra való előfizetéshez. Ez a parancs egy vagy több olyan csatorna nevét várja, amelyen az alkalmazás fogadja az üzeneteket. A StackExchange-kódtár tartalmazza az `ISubscription` illesztőt, amely lehetővé teszi a .NET-keretrendszert használó alkalmazások számára a csatornákra való előfizetést és a rajtuk történő közzétételt.

Az `ISubscription`-objektumot a Redis-kiszolgálóhoz történő kapcsolódás `GetSubscriber` metódusával lehet létrehozni. Ezt követően a csatorna üzeneteit ennek az objektumnak a `SubscribeAsync` metódusával figyelheti. A következő példakód azt mutatja be, hogyan lehet előfizetni a „messages:blogPosts” nevű csatornára:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
await subscriber.SubscribeAsync("messages:blogPosts", (channel, message) => Console.WriteLine("Title is: {0}", message));
```

A `Subscribe` metódus számára megadott első paraméter a csatorna neve. Erre a névre ugyanazok a konvenciók vonatkoznak, mint a gyorsítótár kulcsaira. A név tartalmazhat bármilyen bináris adatot, bár ajánlott viszonylag rövid, jelentéssel bíró sztringekat használni a megfelelő teljesítmény és karbantarthatóság érdekében.

Vegye figyelembe azt is, hogy a csatornák által használt névtér el van választva a kulcsok által használt névtértől. Ez azt jelenti, hogy azonos nevű csatornák és kulcsok is előfordulhatnak, bár ez megnehezítheti az alkalmazáskód karbantartását.

A második paraméter egy Művelet delegált. Ezt a delegáltat a rendszer aszinkron módon lefuttatja, amikor egy új üzenet jelenik meg a csatornán. Ebben a példában az üzenet egyszerűen megjelenik a konzolon (az üzenet tartalmazza a blogbejegyzés címét).

A csatornán történő közzétételéhez az alkalmazások a Redis PUBLISH parancsát használják. Ezen művelet végrehajtásához a StackExchange-kódtár az `IServer.PublishAsync` metódust biztosítja. A következő kódrészlet azt mutatja be, hogyan lehet üzenetet közzétenni a „messages:blogPosts” nevű csatornán:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
BlogPost blogPost = ...;
subscriber.PublishAsync("messages:blogPosts", blogPost.Title);
```

A közzétevői/előfizetői mechanizmus fontos jellemzői:

- Több előfizető is feliratkozhat ugyanarra a csatornára, és mindegyik megkapja az adott csatornán közzétett üzeneteket.
- Az előfizetők csak azokat az üzeneteket kapják meg, amelyeket az előfizetés időpontját követően tettek közzé. A csatornák nem puffereltek, és az üzenet közzétételét követően a Redis-infrastruktúra minden egyes előfizetőnek leküldi az üzenetet, majd eltávolítja azt.
- Alapértelmezés szerint az előfizetők az üzeneteket az elküldés sorrendjében kapják meg. Nagy aktivitást mutató, nagy számú üzenettel, illetve sok előfizetővel és közzétevővel rendelkező rendszerek esetében az üzenetek garantáltan sorrendben történő kézbesítése csökkentheti a rendszer teljesítményét. Ha az üzenetek egymástól függetlenek, és a sorrendjük nem lényeges, engedélyezheti a Redis-rendszer számára az egyidejű feldolgozást, amely révén javítható a válaszképesség. Ez egy StackExchange-ügyfélben úgy valósítható meg, hogy az előfizető által használt kapcsolat PreserveAsyncOrder paraméternél a „hamis” értéket adja meg:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
redisHostConnection.PreserveAsyncOrder = false;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
```

### <a name="serialization-considerations"></a>Szerializálási szempontok

Amikor kiválaszt egy szerializálási formátumot, fontolja meg a teljesítmény, az együttműködési lehetőségek, a verziókezelés, a meglévő rendszerekkel fennálló kompatibilitás, az adatok tömörítése és a memóriaterhelés terén érvényesíthető kompromisszumokat. A teljesítmény kiértékelése során ne feledje, hogy a referenciaértékek nagymértékben függnek a környezettől. Előfordulhat, hogy nem tükrözik a tényleges számítási feladatok hatását, és nem feltétlenül veszik figyelembe az újabb kódtárakat vagy verziókat. Különböző alkalmazási helyzetekben különböző szerializálók bizonyulhatnak a leghatékonyabbnak.

Szóba jöhető lehetőségek:

- [Protokollpufferek](https://github.com/google/protobuf) (más néven protopuf): a Google által kifejlesztett szerializációs formátum a strukturált adatok hatékony szerializálásához. Ez a módszer az üzenetstruktúrák meghatározásához szigorú típusmegadású definíciós fájlokat használ. A rendszer a későbbiekben ezekből a definíciós fájlokból állít össze egy nyelvspecifikus kódot az üzenetek szerializálásához és deszerializálásához. A protopuf a meglévő RPC-mechanizmusokhoz is használható, de egy új RPC-szolgáltatást is létre tud hozni.

- Az [Apache Thrift](https://thrift.apache.org/) hasonló megközelítést alkalmaz, szigorú típusmegadású definíciós fájlokkal és egy fordítási lépéssel a szerializálási kód és az RPC-szolgáltatások létrehozásához.

- Az [Apache Avro](https://avro.apache.org/) funkciói hasonlóak a protokollpufferekhez és a Thrifthez, de hiányzik belőle a fordítási lépés. Ehelyett a szerializált adatok mindig tartalmaznak egy olyan sémát, amelyek leírják a struktúrát.

- A [JSON](https://json.org/) egy felhasználók számára olvasható szövegmezőket használó nyílt szabvány, amely széleskörű, platformfüggetlen támogatást biztosít. A JSON nem használ üzenetsémákat. Mivel szöveges formátumról van szó, a hálózaton keresztüli továbbítása nem túl hatékony. Bizonyos esetekben azonban előfordulhat, hogy a gyorsítótárazott elemeket HTTP-kapcsolaton keresztül küldi vissza közvetlenül az ügyfélnek, amely esetben a JSON formátumban történő tárolással megtakarítható a más formátumról történő deszerializálás, majd a JSON formátumra való szerializálás költsége.

- A [BSON](http://bsonspec.org/) egy, a JSON-hoz hasonló struktúrát használó, bináris szerializálási formátum. A BSON formátum kialakítása során a cél az volt, hogy a JSON-hoz képest kisebb méretű, könnyebben beolvasható, gyorsabban szerializálható és deszerializálható legyen. A hasznos adatok mérete a JSON-éhoz hasonló. Az adatoktól függően a BSON hasznos adatainak mérete kisebb vagy nagyobb is lehet a JSON-énál. A BSON rendelkezik néhány olyan adattípussal, amelyek nem érhetők el a JSON-ban. Ilyen például a BinData (bájttömbökhöz) és a Date (Dátum).

- A [MessagePack](https://msgpack.org/) egy olyan bináris szerializálási formátum, amely kellőképpen kis méretű a hálózaton keresztüli átvitelhez. A MessagePack nem alkalmaz üzenetsémákat vagy üzenettípus-ellenőrzést.

- A [Bond](https://microsoft.github.io/bond/) egy olyan platformfüggetlen keretrendszer, amely sematikus adatok kezelésére lett kifejlesztve. A Bond támogatja a nyelvek közötti szerializálást és deszerializálást. Az itt felsorolt egyéb rendszerekhez képest jelentősebb eltérés, hogy támogatja az öröklést, a típusaliasokat és az általánosítást.

- A [gRPC](https://www.grpc.io/) egy nyílt forráskódú, a Google által kifejlesztett RPC-rendszer. A gRPC alapértelmezés szerint protokollpuffereket használ definíciós nyelvként és az alapul szolgáló üzenetváltási formátumként.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

A következő minták is lehet a forgatókönyvre vonatkozó, amikor az alkalmazások a gyorsítótárazást:

- [Gyorsítótár-feltöltési minta](../patterns/cache-aside.md): Ez a minta ismerteti, hogyan lehet adatokat igény szerint tölthet be egy gyorsítótárba egy adattárolóból. Segítségével arról is gondoskodni lehet, hogy a gyorsítótárban tárolt és az eredeti adattárban lévő adatok konzisztensek maradjanak.

- A [horizontális skálázási minta](../patterns/sharding.md) információkat biztosít a vízszintes particionálás megvalósításáról a méretezhetőség javításához, amikor nagy adatmennyiségeket kell tárolni és hozzáférhetővé tenni.

## <a name="more-information"></a>További információ

- [Az Azure Redis Cache dokumentációja](https://azure.microsoft.com/documentation/services/cache/) 
- [Az Azure Redis Cache – gyakori kérdések](/azure/redis-cache/cache-faq)
- [Feladatalapú aszinkron minta](/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [Redis Cache dokumentációja](https://redis.io/documentation)
- [StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/)
- [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx)
