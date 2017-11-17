---
title: "Megfigyelési és diagnosztikai útmutató"
description: "Gyakorlati tanácsok a felhőben az elosztott alkalmazások figyeléséhez."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 8dd3979233b03db800bd9514263d9c6fedefa074
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="monitoring-and-diagnostics"></a>Megfigyelési és diagnosztikai
[!INCLUDE [header](../_includes/header.md)]

A felhőben futó elosztott alkalmazások és szolgáltatások természetükből, összetett kisebb szoftverek, sok áthelyezése részből áll. Éles környezetben fontos segítségével követheti nyomon, amelyben a felhasználók kihasználhassák a rendszer, az erőforrás-használat nyomkövetési módja, és általában a állapotának és a rendszer teljesítményének figyelése. Segítségével ezt az információt diagnosztikai segítségként ismeri fel és javítsa ki a problémákat, és is potenciális problémák segítségével, és megakadályozza, hogy azok lépett fel.

## <a name="monitoring-and-diagnostics-scenarios"></a>Megfigyelési és diagnosztikai forgatókönyvek
Figyelés segítségével betekintést egy mennyire működik-e a rendszer. A szolgáltatásminőség-tárolók karbantartásának döntő szerepet figyelési. Általános példák a figyelési adatok gyűjtése a következők:

* Győződjön meg arról, hogy a rendszer továbbra is működik megfelelően.
* Nyomon követi, a rendszer és az összetevő elemei.
* Teljesítményét, és győződjön meg arról, hogy az átviteli sebessége a rendszer nem csökkentheti a váratlanul munka növeli a kötetként karbantartása.
* Amely garantálja, hogy a rendszer megfelel-e az ügyfelek létesített szolgáltatásiszint-szerződések (SLA).
* Az adatvédelmi és biztonsági a rendszer, a felhasználók és az adatok védelme.
* A naplózási és szabályozási céljából elvégzett műveletek nyomon követéséhez.
* A rendszer és a trendeket, amely előfordulhat, hogy problémát okozhat, ha még nem érintett gyorsabban napi használat figyelését.
* Nyomon követi a problémák, a lehetséges okok, kijavítása, ezt követő eldobására szoftverfrissítések és központi telepítés elemzésének keresztül kezdeti jelentést.
* Műveletek nyomkövetéséhez és szoftverkiadások.

> [!NOTE]
> Ez a lista nem célja a átfogó. A dokumentum fő témáját forgatókönyvekben, a leggyakoribb esetekben a megfigyelési. Előfordulhat, hogy az adott ritkább vagy adott környezetre.
> 
> 

A következő szakaszok ismertetik részletesebben forgatókönyvekben. Az egyes forgatókönyvek esetében a információkat ismertet a következő formátumban:

1. A forgatókönyv rövid áttekintése
2. Ebben a forgatókönyvben jellemző követelményeinek
3. A forgatókönyv és a lehetséges információforrásokat támogatásához szükséges nyers WMI-adatok
4. Hogyan elemezheti és diagnosztikai információ létrehozásához kombinált a a nyers adatok

## <a name="health-monitoring"></a>Állapotfigyelés
A rendszer állapota kifogástalan, ha fut, és képes a kérelmek feldolgozásához. Állapotfigyelés célja a rendszer aktuális állapotáról pillanatképet létrehozni, így ellenőrizheti, hogy a rendszer minden összetevő várt módon működnek-e.

### <a name="requirements-for-health-monitoring"></a>Állapotfigyelés követelményei
Operátor kell értesítést gyorsan (belül pár másodperc), ha valamely része szerint a rendszer a nem megfelelő kell tekinteni. Az operátor megállapította, a rendszer mely részei megfelelően működnek, és problémákat tapasztal a részeket kell lennie. Rendszerállapot is kiemelten színe rendszeren keresztül:

* A piros sérült (a rendszer leállította)
* A sárga részben megfelelő (a rendszer fut csökkentett)
* A zöld teljesen kifogástalan

Egy átfogó állapotfigyelés rendszer lehetővé teszi, hogy a rendszer alrendszerek és az összetevők állapotának megtekintéséhez részletezés az operátor. Például ha a rendszer általános részben állapota kiváló írja le, az üzemeltető kell tudni nagyítás és annak megállapítása, mely a funkció jelenleg nem érhető el.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
A nyers adatok, állapotfigyelés támogatásához szükséges következtében hozható létre:

* Nyomkövetés felhasználói kérelmek teljesítése. Ezek az információk segítségével határozza meg, mely kérelmek sikeres volt, amelyek nem, és minden kérelem mennyi ideig tart.
* Szintetikus felhasználói figyelése. Ezt a folyamatot a felhasználó által végrehajtott lépések szimulál és a lépések egy előre meghatározott sorozatát követi. Minden lépés rögzíteni kell.
* Kivételek, hibák és figyelmeztetések naplózása. Ezek az információk miatt az alkalmazás kódjának beilleszthető, valamint az Eseménynapló szolgáltatás sem hivatkozik, a rendszer az adatok beolvasása nyomkövetési utasítások rögzíthetők.
* A rendszer külső szolgáltatások állapotának figyelése. Ez a figyelő akkor lehet szükség, beolvasása és elemzése az egészségügyi adatokat, amelyek ezeket a szolgáltatásokat. Ez az információ többféle formátumúak is igénybe vehet.
* Végpontmonitoring kijelző. Ez az eljárás a "Rendelkezésre állásának figyelésére szolgáló" szakaszban részletesen ismertetett.
* Gyűjtése környezeti teljesítményadatok, például a háttér CPU-felhasználás vagy (beleértve a hálózati) i/o-tevékenységet.

### <a name="analyzing-health-data"></a>Rendszerállapot-adatok elemzése
Az állapotfigyelés elsősorban gyorsan jelzi, hogy fut-e a rendszer. A közvetlen gyors adatelemzési is megjelenik egy figyelmeztetés, ha kritikus összetevője észleli megfelelő. (Ez nem válaszol egy ping-üzenetek, egymást követő sorozata például.) Az operátor majd is elvégezheti a szükséges javítási műveleteket.

Egy összetettebb rendszer egy prediktív elem legutóbbi és a jelenlegi munkaterhelés keresztül hajtja végre a cold elemzés közé tartozik. A cold elemzésre tendenciákat, és határozza meg, valószínűleg továbbra is működik megfelelően-e a rendszer, vagy hogy szükséges a rendszer további forrásokat. A prediktív elem alapján kritikus teljesítménymutatók, például:

* Minden szolgáltatás vagy az alrendszer irányuló kérelmek száma.
* Ezek a kérelmek válaszidejét.
* A bejövő és kimenő minden egyes szolgáltatás áramló adatok mennyiségét.

Ha bármely metrika értéke meghaladja a megadott küszöbértéket, a rendszer is riasztást lehetővé teszi, hogy az operátor vagy az automatikus skálázást (ha elérhető) műveletek a megelőző rendszerállapot fenntartásához szükséges. Ezek a műveletek vonatkozhat erőforrások sikertelenek lesznek, és sávszélesség-szabályozás alkalmazása alacsonyabb prioritású kérelmek egy vagy több szolgáltatás újraindítása.

## <a name="availability-monitoring"></a>Rendelkezésre állás figyelése
Egy valóban kifogástalan rendszer igényel, hogy az összetevők és a rendszer alkotó alrendszereket érhetők el. Rendelkezésre állásának figyelésére szolgáló szorosan kapcsolódnak a állapotfigyelés. De állapotfigyelés jeleníti meg egy azonnali a rendszer aktuális állapotát, mivel rendelkezésre állásának figyelésére szolgáló aggasztanak nyomon követése, a rendszer és az összetevőinek a rendszer a megadott hasznos üzemideje statisztikája létrehozásához.

Számos rendszer egyes összetevőket (például egy adatbázis) vannak konfigurálva a beépített redundanciát, így lehetővé teszik a gyors feladatátvétel súlyos hibát vagy a kapcsolat megszakadása esetén. Felhasználók ideális esetben nem kell ügyelnie, hogy az ilyen hiba történt. De figyelési perspektíva rendelkezésre állási, a szükséges legtöbb információt gyűjteni a lehető ilyen a hiba okának megállapításához, és megakadályozhatja, hogy a ismétlődő korrekciós műveletek végrehajtása sikertelen.

Az adatok rendelkezésre állásának nyomon követéséhez szükséges alacsonyabb szintű tényező lehet, hogy függ. Előfordulhat, hogy ezek a tényezők számos adott, az alkalmazás, a rendszer és a környezetben. Hatékony megfigyelési rendszer rögzíti a rendelkezésre állási adatok a alacsony szintű tényezők megfelel-e, majd összesíti őket a rendszer átfogó képet. Például egy e-kereskedelmi rendszerben az üzleti funkció, amely lehetővé teszi az ügyfél rendelések helyezendő előfordulhat, hogy a rendelés részleteit tároló tárház és függ a fizetési rendszer, amely kezeli a pénzügyi tranzakciók ezek rendelések fizet. A rendszer sorrendben-elhelyezés része rendelkezésre állását éppen ezért a tárház és a fizetési alrendszer rendelkezésre állásának függvényében.

### <a name="requirements-for-availability-monitoring"></a>Rendelkezésre állásának figyelésére szolgáló követelményei
Operátor is kell minden egyes rendszer és a alrendszer korábbi rendelkezésre állását, és ezen információk használatával bármely tendenciákat, amely egy vagy több alrendszereket rendszeresen sikertelenségét okozhatja. (Tegye szolgáltatások indítása sikertelen lesz, amely megfelelne a csúcsidőszakon feldolgozási nap adott időpontban?)

Figyelési megoldás az azonnali és korábbi nézetben a rendelkezésre állási vagy minden egyes alrendszer elérhetetlensége kell biztosítania. Is képes gyorsan riasztási operátort, amikor egy vagy több szolgáltatási sikertelen lesz, vagy ha a felhasználók nem tudnak csatlakozni a szolgáltatások. Ez a kérdés nemcsak egyes szolgáltatás figyelése, de is vizsgálata folyamatban van a végrehajtandó műveleteket, minden felhasználó hajt végre, ha ezek a műveletek sikertelenek, amikor azok megkísérlik a szolgáltatással való kommunikációra. Bizonyos mértékig csatlakozási hiba fokú normális, és átmeneti hibák okozták lehet. De a rendszer riasztást kapcsolathibái megadott alrendszerhez egy adott időszakban fellépő számának hasznos lehet.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
Csakúgy, mint a állapotfigyelés, a nyers adatok rendelkezésre állásának figyelésére szolgáló támogatásához szükséges figyelés és naplózás, bármilyen kivételek, hibák és figyelmeztetések, amely akkor jelentkezhet szintetikus felhasználói miatt hozhatók létre. Emellett a rendelkezésre állási adatok szerezhető be végpont megfigyelési. Az alkalmazás is elérhetővé teheti, egy vagy több állapotfigyelő végpontot, egy funkcionális területen belül a rendszer minden egyes tesztelési eléréséhez. A felügyeleti rendszer minden egyes végpont ping meghatározott ütemezés szerint, és az eredmények (sikeres vagy sikertelen) gyűjtése.

Az összes időtúllépések, hálózati kapcsolathibái és kapcsolódási próbálkozást rögzíteni kell. Minden adat időbélyeggel ellátott kell lennie.

<a name="analyzing-availability-data"></a>

### <a name="analyzing-availability-data"></a>Rendelkezésre állási adatok elemzése
A WMI-adatok kell összesíteni és korrelált elemzés a következő típusú támogatásához:

* A közvetlen rendelkezésre állását, valamint a rendszer alrendszereket.
* A rendelkezésre állási hiba sebességét, a rendszer és alrendszereket. Ideális esetben operátor hibák okozták az adott tevékenységek képesnek kell lennie: Mi zajlik, amikor a rendszer nem tudta?
* Hiba sebességű a rendszer vagy bármely alrendszerek közötti bármely ügyfélállapotainak megadott időszak, és a terhelés a rendszeren (például a felhasználói kérelmek száma) Ha a hiba történt.
* A rendszer vagy bármely alrendszereket elérhetetlensége okait. Például lehet, hogy a okok szolgáltatás nem fut, a kapcsolat megszakadt, de időtúllépés és a csatlakoztatott, de az adatszolgáltató hibák csatlakoztatva.

Egy meghatározott időtartamra vonatkozóan szolgáltatás százalékos rendelkezésre állását a következő képlettel számolhatja ki:

```
%Availability =  ((Total Time – Total Downtime) / Total Time ) * 100
```

Ez akkor hasznos, SLA célokra. ([SLA figyelési](#SLA-monitoring) Ez az útmutató későbbi részében részletesebben leírt.) Definíciója *állásidő* a szolgáltatástól függ. Például a Visual Studio Team Services Build szolgáltatás állásidő meghatározása szerint az időszak (halmozott percek) számlálási időintervalluma Build szolgáltatás nem érhető el. Egy percet akkor tekinthető nem érhető el, ha a folyamatos HTTP-kérelmek szolgáltatás létrehozása során a percet ügyfél által kezdeményezett műveletek végrehajtásához egy hibakódot eredményezhet, vagy nem válaszol.

## <a name="performance-monitoring"></a>Teljesítményfigyelés
A rendszer helyezkedik el, egyre több magas terhelés alatt (a felhasználók a kötet növelésével), ezek a felhasználók hozzáférést növekedésének adathalmaz méretének és a lehetőségét, amely egy vagy több összetevő hibája válik nagy valószínűséggel. Gyakran összetevő meghibásodása esetén a teljesítmény csökkenését előzi meg. Ha elvégezheti észleli a csökkenést, elvégezhető a helyzet orvoslásához proaktív lépéseket.

A rendszer teljesítménye számos tényezőtől függ. Minden egyes tényező általában mérik fő teljesítménymutatók (KPI), például adatbázis-tranzakciók másodpercenkénti számát vagy a kötet hálózati kérelmek, amelyet sikeresen egy megadott időkereten belül. Ezen KPI-k némelyike előfordulhat, hogy használhatók az adott intézkedéseket, mivel előfordulhat, hogy a metrikák kombinációja származtatni mások számára.

> [!NOTE]
> Gyenge vagy a jó teljesítmény meghatározásához szükséges, hogy tudomásul veszi a szintű teljesítmény, ahol a rendszer fut képesnek kell lennie. Ehhez a megfigyelő a rendszer, míg az átlagos terhelés mellett működik, és az adatok rögzítésével az egyes KPI-KET egy meghatározott időtartamra vonatkozóan. Ez lehet, hogy magában, a rendszer egy szimulált terhelés alatt fut egy tesztkörnyezetben, és a megfelelő adatokat gyűjt a rendszer egy éles környezetbe történő telepítése előtt.
> 
> Gondoskodnia kell arról, hogy a teljesítmény célokra figyelés nem lesz terhet a rendszeren. Előfordulhat, hogy dinamikusan úgy, hogy a Teljesítményfigyelő folyamat összegyűjti az adatok részletességi szintje.
> 
> 

### <a name="requirements-for-performance-monitoring"></a>Teljesítményfigyelés követelményei
Vizsgálja meg a rendszer teljesítményét, operátor általában kell megfelelnie is információkat lásd:

* A felhasználói kérelmek adott válasz sebességet.
* Az egyidejű felhasználói kérelmek száma.
* A hálózati forgalom mennyisége.
* A sebesség, mely üzleti tranzakciók vannak befejezését.
* Az átlagos feldolgozási ideje a kéréseket.

Azt is hasznos lehet a adja meg az eszközök, amelyek lehetővé teszik a helyszíni korrelációk segítségével operátor:

* A kérés várakozási ideje időpontokban (mennyi ideig tart egy kérelem feldolgozása után a felhasználó rendelkezik az indításához) és az egyidejű felhasználók száma.
* Az átlagos válaszidő (mennyi ideig tart a kérelem végrehajtása után feldolgozása megkezdődött) és az egyidejű felhasználók száma.
* A mennyiségű kérést, és a feldolgozási hibák száma.

A magas szintű működési információk mellett egy olyan operátort kell tudni teljesítménye az egyes összetevők részletes képet kaphat a rendszerben. Ezeket az adatokat általában nyomon követheti, mint az alsó szintű teljesítményszámlálók keresztül elérhető:

* Memóriafelhasználás.
* Szálak száma.
* Processzor feldolgozási ideje.
* Kérelmek várólistájának hossza.
* Lemez- vagy i/o-sebességét és hibákat.
* Ennyi bájtot írt, vagy olvassa el.
* Köztes mutatókat, például a várólista hossza.

Minden képi megjelenítés lehetővé kell tennie az üzemeltető számára adjon meg egy időszakot. A megjelenített adatok a jelenlegi szituáció pillanatképe és/vagy a teljesítmény ügyfélállapotainak lehet.

Egy olyan operátort kell tudni hoz létre riasztást, a megadott időintervallumon bármely teljesítménymérték azok számát a megadott érték alapján.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
A felhasználói kérelmek állapotának figyelésével, még az ügyfélszámítógépekre érkeznek, és továbbítja a rendszer gyűjthet magas szintű teljesítményadatok (átviteli, egyidejű felhasználók száma, az üzleti tranzakciók, hiba díjszabás és így tovább). Ez magában foglalja a nyomkövetési adatokkal időzítése együtt az alkalmazáskódban található kapcsolatos legfontosabb utasításokat tartalmazó. Minden hibák, kivételek és figyelmeztetések használatával történik, azokat a kérelmeket, ami miatt azokat a megfelelő adatokkal lesznek rögzítve. Az Internet Information Services (IIS) napló egy másik hasznos forrásaként szolgál.

Ha lehetséges a külső rendszerekkel, az alkalmazás által használt teljesítményadatokat is rögzíti. A külső rendszerekből előfordulhat, hogy adja meg a saját teljesítményszámlálók és egyéb szolgáltatások ügynökteljesítmény-adatokat kér. Ha ez nem lehetséges, rekord információkat, például a kezdési és befejezési időpontja, a művelet állapotát (sikeres, sikertelen vagy figyelmeztetés) együtt a külső rendszerek minden kérelmet. Például használhatja a stoppert megközelítés ideje kérésekre: indításakor időzítő a kérelmet, és állíthatja le a időzítőt, ha a kérelem végrehajtása.

Lehet, hogy a rendszer az egyes összetevők alacsony szintű teljesítményadatokat funkciókat és szolgáltatásokat, például Windows-teljesítményszámlálók és az Azure Diagnostics elérhető.

### <a name="analyzing-performance-data"></a>A teljesítményadatok elemzése
Munka nagy része a elemzés áll összesítése a felhasználói kérelem típusa és/vagy az alrendszer vagy a minden kérelmet küldenek szolgáltatás teljesítményadatait. A felhasználó kérésére például cikk hozzáadása egy kosár vagy a kivételt folyamat végrehajtása egy e-kereskedelmi rendszerben.

Egy másik közös követelmény kijelölt százalékos érték teljesítményadatok kivonatolását végzi. Operátor például előfordulhat, hogy meghatározzák a válaszidők kérelmek százalékos 99, a kérelmek 95 %-os és a kérelmek 70 százalékos. Előfordulhat, hogy SLA célokat, vagy minden érték más célok állítani. A folyamatban lévő eredményeit azokat a problémák azonnali közel valós idejű kell megadni. Az eredmények is statisztikai célokra hosszabb idő lehet összesíteni.

Esetén késési problémák teljesítményét operátor gyorsan azonosíthatja a szűk keresztmetszetek okát a várakozási minden egyes lépést minden kérést végrehajtó megvizsgálásával kell lennie. A teljesítményadatok ezért biztosítania kell a teljesítmény mérőszámai egy adott kérelem az kelljen egyes lépéseihez szükséges adatok módszert.

A képi megjelenítés követelményeitől függően lehet hasznos, ha szeretné létrehozni és tárolni egy adatkockát, amely a nyers adatok nézeteinek tartalmazza. Az adatkocka lehetővé tehetik a összetett alkalmi kérdez le, és a teljesítményadatok elemzése.

## <a name="security-monitoring"></a>A biztonság ellenőrzése
Bizalmas adatokat tartalmazó összes kereskedelmi rendszereket meg kell valósítania egy biztonsági struktúra. A biztonsági mechanizmust összetettsége az általában az adatok érzékenységének függvényében. A felhasználók hitelesítését igénylő rendszer rögzíteni kell:

* Az összes kísérlet, hogy azok sikertelen vagy sikeres.
* Minden műveletet végzi--, és által--hitelesített felhasználók elért összes erőforrás részleteit.
* Ha a felhasználó a munkamenet véget ér, és kijelentkezésekor lép érvénybe.

Figyelési lehet segítségével észlelheti a a rendszeren. Például a nagy számú sikertelen bejelentkezési kísérletek találgatásos támadásra utalhat. Egy váratlan megugrását kérésekben okozhatja egy elosztott-szolgáltatásmegtagadásos (DDoS-) támadások. Fel kell készülni figyelheti az összes kérelmet, függetlenül attól, ezek a kérelmek forrását összes erőforrást. A rendszer, amely rendelkezik a bejelentkezési biztonsági rés véletlenül esetleg felfedi a külvilág erőforrásokat anélkül, hogy a felhasználót, hogy ténylegesen jelentkezzen be.

### <a name="requirements-for-security-monitoring"></a>Biztonsági figyelés követelményei
A legfontosabb szempontok a biztonsági figyelés gyorsan engedélyezze az operátor:

* Megkísérelt behatolások észleléséhez egy nem hitelesített entitás.
* Kísérletek azonosíthatja az adatokat, amelyekre nem lett hozzáféréssel rendelkeznek a műveletek végrehajtásához entitásokat.
* Megállapításához, hogy a rendszer, vagy a rendszer, egyes részei kívül vagy belül a támadás alatt áll. (Például egy rosszindulatú hitelesített felhasználó előfordulhat, hogy próbál leállíthatja a rendszer.)

Ezek a követelmények támogatása érdekében értesíteni kell, operátor:

* Ha egy fiók ismételt sikertelen bejelentkezési kísérletek egy adott időszakon belül.
* Ha egy hitelesített fiók ismételten megpróbál tiltott erőforrások eléréséhez a meghatározott időszakon belül.
* Ha a nem hitelesített vagy jogosulatlan kérelmek nagy számú meghatározott időszakon belül történik.

A kezelőként megadott információkat kell tartalmaznia a gazdagép címe a forrás-az egyes kérelmek. Ha biztonsági problémát rendszeresen egy adott címtartományból merülnek fel, ezek a gazdagépek blokkolhatja.

A rendszer biztonságának fenntartásában kulcs része gyorsan észleli, amelyek eltérnek a szokásos mintát műveletek alatt. Például a sikertelen vagy sikeres bejelentkezési kérelmek száma észleli, ha van egy csúcs tevékenységben szokatlan időpontban segítségével vizuálisan megjeleníthető. (Ez a tevékenység példa: 3:00 órakor bejelentkezés, és számos műveletek elvégzésére, 9:00 órakor a munkanapot indításakor felhasználók). Ez az információ is használható időalapú automatikus skálázás konfigurálása érdekében. Például ha egy olyan operátort figyelembe veszi, hogy a felhasználók sok rendszeresen jelentkezzen be a nap adott időpontban, az üzemeltető rendezheti elindítani a további hitelesítési szolgáltatásokat, és a munka mennyiségét kezelésére, majd állítsa le a további szolgáltatások esetén a maximális a rendszer megfelelt.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
A biztonság az legtöbb elosztott rendszerek minden felölelő szempontját. A lényeges adat valószínű, hogy a rendszer több pontokon hozható létre. Érdemes lehet összegyűjteni a biztonsági események emelt az alkalmazás, a hálózati berendezések, a kiszolgálók, a tűzfalak, a víruskereső szoftver, és más által a biztonsági adatai és az esemény felügyeleti SIEM-megközelítés bevezetése behatolás-megelőzési elemek.

Biztonsági figyelési eszközök, amelyek nem szerepelnek az alkalmazás adatait építhessék be. Ezek az eszközök segédprogramok, amely azonosítja a külső szervezetek, vagy az alkalmazás és az adatok nem hitelesített hozzáférést irányuló kísérletek hálózati szűrők port vizsgálatát tevékenységeket tartalmazhatnak.

Minden esetben az összegyűjtött adatokat a rendszergazda határozza meg a támadás természetét és tegye meg a megfelelő ellenintézkedések engedélyeznie kell.

### <a name="analyzing-security-data"></a>Biztonsági adatok elemzése
A biztonsági figyelés jellemzője, a különböző forrásokból, amelyből az adat keletkezik. A különböző formátumokban és részletességi szintje gyakran megkövetelik a összetett elemzésének alkalmazássá be adatok egy összefüggő szál rögzített adatokat. Mellett a legegyszerűbb esetekben (például nagyszámú sikertelen bejelentkezések, vagy a kritikus fontosságú erőforrásokhoz való jogosulatlan hozzáférést ismételt kísérletek észlelése) hogy előfordulhat, hogy nem lehet végrehajtani a biztonsági adatok bármely összetett az automatikus feldolgozása. Ehelyett lehet érdemes időbélyeggel ellátott egyéb szakértői manuális elemzéshez engedélyezéséhez biztonságos tárházba eredeti formájukban, de ezek az adatok írása.

<a name="SLA-monitoring"></a>

## <a name="sla-monitoring"></a>SLA-figyelés
Fizető vevők támogató sok kereskedelmi rendszerek garanciák kapcsolatban a rendszer teljesítményét ellenőrizze SLA-k formájában. Alapvetően SLA-k állapotban, hogy a rendszer kezelni tud a meghatározott mennyiségű munka az egyeztetett időkereten belül, és a kritikus adatok elvesztése nélkül. Győződjön meg arról, hogy a rendszer megfelel mérhető SLA SLA figyelési érintett.

> [!NOTE]
> SLA-figyelési szorosan kapcsolódnak a teljesítmény figyelése. De mivel teljesítményfigyelés van biztosítva, hogy a rendszer működését *optimális*, SLA figyelési szabályozza, amely meghatározza, milyen szerződéses kötelezettség *optimális* ténylegesen azt jelenti, hogy.
> 
> 

SLA-k a definiálja:

* Általános rendelkezésre állására. Például egy szervezet előfordulhat, hogy garantálja, hogy a rendszer 99,9 %-ában elérhető lesz-e. Ez megfelel a legfeljebb 9 óra állásidő évente, vagy a hét körülbelül 10 perc.
* Működési átviteli sebességet. Egy vagy több magas vízjeleket, amely garantálja, hogy a rendszer legfeljebb 100 000 egyidejű felhasználói kérelmek támogatásához, vagy 10 000 egyidejű üzleti tranzakciók kezelése például gyakran kifejezése ezen tulajdonsága.
* Működési válaszidő. A rendszer is tehetik a kérelmek feldolgozásának sebessége garanciát. Például, hogy minden üzleti tranzakciók 99 százalékának befejezi 2 másodpercen belül, de nincs egyetlen tranzakció 10 másodpercnél hosszabb ideig tart.

> [!NOTE]
> Néhány szerződések kereskedelmi rendszerekhez kiterjedhetnek SLA-k az ügyfél-támogatási szolgálatához. Például, hogy minden segélyszolgálat kérelmeket 5 percen belül fog kér választ, és hogy az összes probléma 99 %-át teljes kiiktatása 1 munkanapon belül. Hatékony [problémák nyomon követése](#issue-tracking) (később ebben a szakaszban leírt) van szolgáltatásiszint-szerződéseknek ehhez hasonló helyzeteknek billentyűt.
> 
> 

### <a name="requirements-for-sla-monitoring"></a>SLA-figyelés követelményei
A legmagasabb szintjén az üzemeltető meg tudja határozni egy pillanat alatt, hogy a rendszer megfelel-e a megállapodás szerinti SLA-k vagy nem kell lennie. Ha nem, a következő operátor kell tudni részletezéshez és le, és megvizsgálja a mögöttes tényezők állapítsa meg, hogy a követelményeket teljesítmény.

Tipikus magas szintű mutatók vizuálisan kitaláltak a következők:

* Hasznos üzemidő százalékát.
* Az alkalmazás az átviteli sebesség (sikeres tranzakciók és/vagy a műveletek másodpercenkénti száma).
* Sikeres/sikertelen alkalmazás kérések száma.
* Az alkalmazás- és hibák, kivételek és figyelmeztetések száma.

Ezek a mutatók mindegyikét a megadott időtartam szűri képesnek kell lennie.

A felhőalapú alkalmazások valószínűleg magában foglalja a több alrendszerek és összetevőket. Operátor válassza ki a magas szintű kijelzőt, és tekintse meg, hogyan össze az alapul szolgáló elemek tartozó képesnek kell lennie. Például ha a teljes rendszer üzemideje elfogadható érték alá csökken, egy olyan operátort kell tudni nagyítás, és döntse el, melyik elemet biztosítanak a hiba.

> [!NOTE]
> Rendszer hasznos üzemidő kell gondosan definiálni. A maximális rendelkezésre állásának biztosításához redundancia használó rendszer elemek egyes példányai sikertelen lehet, de maradhat, hogy a rendszer működési. Állapotfigyelés által bemutatott rendszer üzemideje fel kell tüntetni az egyes elemek összesített üzemideje és nem feltétlenül e a rendszer ténylegesen leállt. Emellett előfordulhat, hogy lehet elkülönített hibák. Így akkor is, ha egy megadott rendszer nem érhető el, a rendszer további része lehet, hogy elérhetők maradnak, bár a csökkent funkcionalitással. (Egy e-kereskedelmi rendszerben hibája miatt a rendszer az ügyfél esetleg nem rendelések helyezi el, de előfordulhat, hogy a az ügyfél továbbra is el tud jutni a termékkatalógus.)
> 
> 

Riasztások céljából, a rendszer kell tudni indítson eseményt, ha a magas szintű mutatók bármelyikét haladja meg a megadott küszöbértéket. A számos tényező befolyásolja, a magas szintű kijelző alkotó alacsonyabb szintű részleteit a riasztási rendszer környezetfüggő adatként elérhetőnek kell lennie.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
A nyers adatok SLA figyelési támogatásához szükséges hasonlít a nyers adatok teljesítményfigyelési, állapotának és rendelkezésre állásának figyelésére szolgáló bizonyos aspektusainak együtt van szükség. (Ezek a szakaszok további részletekért tekintse meg.) Ezek az adatok által rögzítése:

* Végez a végpontmonitoring kijelző.
* Kivételek, hibák és figyelmeztetések naplózása.
* A felhasználói kérelmek teljesítése nyomkövetés.
* A rendszer által használt külső szolgáltatások rendelkezésre állásának figyelése.
* Metrikák és számlálók használatával.

Minden adat legyen túllépte az időkorlátot és időbélyeggel ellátott.

### <a name="analyzing-sla-data"></a>SLA-adatok elemzése
A WMI-adatok létrehozására, a rendszer általános teljesítménye képe kell összesíteni. Összesített adatokat is támogatnia kell a Részletezés engedélyezése az alapul szolgáló alrendszerek teljesítmény vizsgálata. Például meg kell tudni:

* A felhasználói kérelmek teljes száma kiszámítása a meghatározott időszakon belül, és ezeket a kéréseket a sikeres és sikertelen arányát határozza meg.
* Összevonja a felhasználói kérelmek rendszer válaszidejének átfogó képet létrehozásához a válaszidők.
* Vizsgálja meg a felhasználói kérelmek szeretné bontani a kérelemben található egyedi munkaelemeket válaszidő kérelem teljes válaszidő előrehaladását.  
* Hasznos üzemidő százalékban a rendszer általános rendelkezésre állására meghatározása egy adott időszakban.
* A százalékos idő rendelkezésre állását az egyes összetevőkkel és szolgáltatásokkal elemzése a rendszerben. Ez lehet, hogy tartalmaz, amely harmadik féltől származó szolgáltatással létrehozott naplók elemzése.

Számos kereskedelmi rendszer valós teljesítmény ábra elleni egyeztetett SLA jelentés egy megadott időtartamig, általában egy hónap szükségesek. Ezek az információk segítségével kiszámítani kreditek vagy egyéb visszafizetések az ügyfelek Ha adott időszakban nem felel meg az SLA-k. A szolgáltatás rendelkezésre állásának kiszámíthatja a szakaszban bemutatott módszerrel [rendelkezésre állási adatok elemzése](#analyzing-availability-data).

Belső célra egy szervezet előfordulhat, hogy is nyomon követheti az a szám és a szolgáltatások nem fognak kiváltó incidensek jellege. A problémák megoldásához gyorsan, vagy kiküszöbölheti a teljesen, hogyan csökkentheti az állásidőt, és megfelelnek a szolgáltatásiszint-szerződések segítségével.

## <a name="auditing"></a>Naplózás
Az alkalmazás természetétől függően előfordulhat, kötelező vagy egyéb jogszabályok által megadott felhasználói műveletek naplózása és az összes adatelérési rögzítése követelményei. Naplózás is bizonyító, hogy hivatkozásokat a felhasználók bizonyos kérésekre. A nem megtagadás számos üzleti e rendszer megbízhatósági fenntartása érdekében fontos tényezőt kell az ügyfél és a szervezet, amely felelős az alkalmazás vagy szolgáltatás között.

### <a name="requirements-for-auditing"></a>Naplózásának követelményei
Valamelyik elemzőnek, hogy a felhasználók így is újbóli összeállítása a felhasználói műveleteket hajt végre üzleti műveletek sorrendjének követéséhez képesnek kell lennie. Erre akkor lehet szükség, egyszerűen alkot rekordot, vagy a törvényszéki nyomozások részeként.

A naplóinformációkat szigorúan bizalmas. Ez valószínűleg magában foglalja, amely azonosítja a felhasználókat a rendszer a feladatokat helyreállítást hajt végre, és adatokat. Emiatt a naplóinformációkat valószínűleg a következőkben csak megbízható elemzők rendelkezésre álló jelentések helyett egy interaktív rendszert, amely támogatja a grafikus műveletek részletes. Valamelyik elemzőnek számos különböző jelentések létrehozásához képesnek kell lennie. Például jelentések előfordulhat, hogy minden felhasználó tevékenységek egy adott időtartományban listában, a tevékenység egyetlen felhasználóhoz időrendje részletességi vagy listában egy vagy több erőforrásokon végrehajtott műveletek sorozata.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
Az elsődleges információforrások a naplózáshoz az alábbiakból állhat:

* A biztonsági rendszer, amely kezeli a felhasználó hitelesítése.
* Nyomkövetési naplók, amely a felhasználó tevékenységét rögzíti.
* Minden azonosítható és azonosítatlan hálózati kérelmeket nyomon biztonsági naplókat.

A formátum, amely tárolja a naplózási adatok és szabályozási követelmények alapján előfordulhat, hogy vezeti. Például akkor lehet, hogy nem lehet bármely olyan módon adatot. (Ez rögzíteni kell az eredeti formátumában.) A tárház tárolási helye a hozzáférést az illetéktelen kell védeni.

### <a name="analyzing-audit-data"></a>Naplózási adatok elemzése
Lehet, hogy valamelyik elemzőnek érhessék el az eredeti formájukban egészében nyers adatok. A követelmény a gyakori naplózási jelentések készítéséhez, vezérelt ezek az adatok elemzésére szolgáló eszközöket valószínűleg kifejezetten és a rendszer a külső tartani.

## <a name="usage-monitoring"></a>Használat figyelését
Használat figyelését nyomon követi a szolgáltatásai és összetevői az alkalmazások használatának módját. Operátor használhatja az összegyűjtött adatokat:

* Határozza meg, mely funkciókat fokozottan használatosak, és a rendszer minden lehetséges elérési pontokhoz történő meghatározásához. Nagy forgalmú elemek előnye származhat működési particionálás és a terhelés több egyenletesen még akkor is, replikációt. Operátor is használhatja ezt az információt megállapítani, mely funkciókat ritkán használja, és lehetséges alternatívák vagy cserélni a rendszer egy jövőbeli verziójában.
* A rendszer normál használják a működési események információhoz juthat. Például az elektronikus kereskedelmi webhely, rögzítheti a tranzakciók száma és az ügyfelek, akik felelős a kötet statisztikai adatait. Ezt az információt a kapacitástervezés ügyfelek számának növekedésével használható.
* Észleli az (esetleg közvetetten) felhasználói számára a teljesítmény vagy a rendszer működését. Például ha az ügyfelek egy e-kereskedelmi rendszerben nagy számú rendszeresen abandon a vásárlási kártyák, ennek oka lehet a vegye ki funkciójú probléma.
* Készítése a számlázási adatokat. Egy kereskedelmi alkalmazás vagy a több-bérlős szolgáltatást a előfordulhat, hogy az ügyfelek az általuk használt erőforrások díjat számítanak.
* Kvóták kényszerítéséhez. Ha egy felhasználó egy több-bérlős rendszerben meghaladja a feldolgozási idő vagy az erőforrás-használat egy meghatározott időtartam alatt a fizetős kvótát, a hozzáférés korlátozható vagy feldolgozási is szabályozva.

### <a name="requirements-for-usage-monitoring"></a>Használat figyelését követelményei
Vizsgálja meg a rendszer használati, operátor általában kell megfelelnie is információkat lásd:

* Minden egyes alrendszer által feldolgozott, és az egyes erőforrások felé irányuló kérelmek száma.
* A munkát végző minden felhasználóhoz.
* A kötet, amely minden felhasználó által elfoglalt adattárolási.
* Az erőforrások minden felhasználó hozzáférhet.

Operátor hozhat létre diagramokat kell. Például egy grafikonon megjelenítheti a legtöbb erőforrást belekóstol felhasználókat, vagy a leggyakrabban elért erőforrások vagy -funkciókat.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
Használat nyomon követése viszonylag magas szinten hajtható végre. Azt is vegye figyelembe, az egyes kérelmek indításának és befejezésének idejét és a kérés (olvasási, írási és így tovább, attól függően, hogy az adott erőforrás) jellegét. Ezt úgy szerezheti be ezeket az információkat:

* Felhasználói tevékenység nyomkövetés.
* A rögzítés teljesítményszámlálók találhatók, amelyek mérik az egyes erőforrások kihasználtsága.
* Az erőforrás-felhasználás, mely felhasználó figyelése.

Mérés céljából, is kell lennie lehet azonosítani, hogy mely felhasználók felelőssége milyen műveleteket, és az erőforrásokat, amelyek használják ezeket a műveleteket hajt végre. Az összegyűjtött információk kell lennie ahhoz, hogy pontos számlázási részletes.

<a name="issue-tracking"></a>

## <a name="issue-tracking"></a>A problémák nyomon követése
Az ügyfelek és más felhasználók előfordulhat, hogy jelenti problémák esetén nem várt események vagy viselkedés a rendszerben. Problémák nyomon követése a problémák kezelése, való azon törekvéseit, hogy a rendszer minden alapul szolgáló problémák megoldásához és a lehetséges megoldások a felhasználók tájékoztatása tekintetében.

### <a name="requirements-for-issue-tracking"></a>A problémák nyomon követése követelményei
Operátorok gyakran hajtsa végre a problémák nyomon követése külön rendszer-, amely lehetővé teszi, hogy rekord és a jelentés a problémák részleteit, amelyek felhasználók jelentést készítenek. Ezek az adatok tartalmazhatnak a feladatokat, a felhasználó próbál végrehajtani, a problémát, eseménysorozatát, és semmilyen hiba vagy kiadott figyelmeztető üzeneteket.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
Probléma-nyomon követési adatok kezdeti adatforrását az a felhasználó, aki a problémát az elsőként jelentett. Lehet, hogy a felhasználó további adatokat nyújt, mint:

* Összeomlási memóriaképet (ha az alkalmazás a felhasználói asztali környezetben futtató összetevőt tartalmazza).
* Képernyő pillanatképet.
* A dátum és idő, amikor a hiba történt, például a felhasználó földrajzi helye más környezeti adatokkal együtt.

Ez az információ használható a hibakeresési elérhető, és a későbbi kiadásokban a szoftverfrissítés Hátralék összeállításához.

### <a name="analyzing-issue-tracking-data"></a>Probléma-nyomkövetési adatok elemzése
Különböző felhasználók előfordulhat, hogy a probléma jelenti. A probléma követőrendszerrel közös jelentések kell társítani.

A hibakeresési elérhető előrehaladását feljegyzik minden probléma jelentés ellen. Ha a probléma megoldódott, az ügyfél is tájékoztatni a megoldás.

Ha a felhasználó azt jelenti, amely rendelkezik egy ismert megoldás a probléma követőrendszerrel problémát, a következő operátor kell tudni azonnal tájékoztatja a felhasználót, a megoldás.

## <a name="tracing-operations-and-debugging-software-releases"></a>Műveletek szoftver verziókban nyomkövetéséhez és
A felhasználó problémát jelent, ha a felhasználó gyakran csak tisztában a közvetlen hatással lehet, hogy azok a műveletek az. A felhasználó is csak jelentik az eredményeket a saját felhasználói felület vissza egy olyan operátort, aki felelős a rendszerben. Ezek a tapasztalatok rendszerint látható tünete egy vagy több alapvető problémák. Sok esetben egy elemző kell átrágniuk a időrendje az alapul szolgáló műveletek a probléma okának meghatározásához. Ez a folyamat *kiváltó okot*.

> [!NOTE]
> Legfelső szintű okát előfordulhat, hogy fedik le a hatékonyság hiánya az alkalmazások kialakításában. Ezekben a helyzetekben esetleg az érintett elemek átdolgozási, és központi telepítésére egy későbbi kiadásában. Ez a folyamat gondos hozzáférésre van szüksége, és a frissített összetevők szorosan kell figyelni.
> 
> 

### <a name="requirements-for-tracing-and-debugging"></a>Nyomkövetés és hibakeresés vonatkozó követelmények
A nyomkövetés nem várt események és egyéb problémák, létfontosságú, hogy a figyelési adatokat biztosít-e elegendő információt egy elemző nyomkövetéshez vissza ezeknek a problémáknak a források az engedélyezése, és hozza létre újból az események, sorozatát. Ez az információ ahhoz, hogy valamelyik elemzőnek oka az esetleges problémák diagnosztizálásához elegendőnek kell lennie. A fejlesztők is végezze el a szükséges módosításokat, megakadályozhatja, hogy a ismétlődő.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Az adatforrások, instrumentation és adatgyűjtés követelmények
Hibakeresési nyomkövetés összes módszert (és a paraméterek) meghívni, amely a rendszeren keresztül tételéig ábrázol, amikor egy ügyfél egy adott kérelmet fát kialakításához művelet részeként magába foglaló. Rögzített és naplózott kell kivételeket és a figyelmeztetéseket, mivel ez a folyamat során a rendszer létrehozza.

Támogatja a hibakeresés, a rendszer képes hurkok, amelyek lehetővé teszik a rendszer kritikus fontosságú pontokon állapotadatokat rögzítéséhez operátor. Vagy a rendszer által biztosított kijelölt műveletek állapotát részletes lépéseit. Ezen a szinten részletességi rögzítésével adatok adhat meg a rendszer további terhelést, és egy ideiglenes folyamat kell lennie. Operátort használja ezt a folyamatot, elsősorban akkor, ha nagyon szokatlan számos esemény következik be, és nehéz replikálni, vagy ha egy új a egy vagy több eleme rendszerben verzióhoz gondos figyelése annak érdekében, hogy a várt módon elemek függvény.

## <a name="the-monitoring-and-diagnostics-pipeline"></a>A figyelés és diagnosztika-feldolgozási folyamat
Nagy méretű elosztott a figyelést jelentős kihívást jelent. Minden az előző szakaszban leírt forgatókönyv nem feltétlenül tekintendő elkülönítve. Nincs feltehetőleg egy jelentős átfedésben lévő, amely az egyes esetekben szükséges megfigyelési és diagnosztikai adatokat, de ezek az adatok feldolgozása és jelenik meg a különböző módokon lehet, hogy kell-e. Ezen okok miatt kell venni a figyelési és diagnosztika átfogó képet.

A teljes figyelése és diagnosztika folyamat is előirányozni, egy folyamatot, amely magában foglalja a szakaszt, 1. ábrán látható.

![A figyelés és diagnosztika feldolgozási szakaszból](./images/monitoring/Pipeline.png)

*1. ábra. A figyelés és diagnosztika feldolgozási szakaszból*

1. ábra mutatja be, hogyan figyelési és diagnosztikai adatokat az adatforrások különböző származhatnak. A rendszerállapot és a gyűjtemény szakaszában, amikor az adatok kell-e rögzíteni kell, hogy az adatforrások azonosító van szó, mely adatok rögzítéséhez, azt rögzítése és formázása ezeket az adatokat, így könnyen vizsgálni meghatározásához. Az elemzés/diagnosztikai szakasza a nyers adatokat fogad, és lekérdezhetik a fontos információkat, amelyek operátor használatával határozhatják meg, a rendszer állapotának létrehozásához használja. Az üzemeltető lehetséges műveletek kapcsolatos döntések érvénybe ezen információk használatával, és majd hírcsatorna újra üzembe a rendszerállapot és a gyűjtemény szakaszában az eredményeket. A képi megjelenítés/riasztások szakaszban fázis a rendszerállapot fogyasztható nézetét mutatja be. Azt is megjelenítheti közel valós idejű irányítópultokat sorozatának használatával. És az hozhat létre jelentéseket, a diagramok és a diagram az adatok, amelyek segítségével azonosíthatja a hosszú távú előzményadatok áttekintése. Ha információt azt jelzi, hogy KPI valószínűleg hosszabb legyen, mint a elfogadható határai, ebben a szakaszban is elindítható operátor riasztást. Bizonyos esetekben egy riasztás is is indításához használható egy automatikus folyamat, amely megpróbálja korrekciós műveletek, például az automatikus skálázást.

Vegye figyelembe, hogy ezeket a lépéseket egy folyamatos üzenetkezelési folyamat, amelyben párhuzamosan a szakaszok történik hoz létre. Ideális esetben a fázisok dinamikusan konfigurálható legyen. Bizonyos időpontokban különösen akkor, ha a rendszer újonnan telepítve van, vagy problémákba ütközött, szükség lehet gyakrabban a kiterjesztett adatok gyűjtéséhez. A többi időszakban állítható vissza egy alap szint, győződjön meg arról, hogy a rendszer megfelelően működik-e a lényeges információk rögzítése lehetővé kell tenni.

Ezenfelül a teljes felügyeleti folyamatot figyelembe kell venni egy élő, folyamatosan megoldás, amely pontosabb beállításra és visszajelzés miatt fejlesztései. Például előfordulhat, hogy a kiindulási pont rendszer állapotának számos tényező méri. Időbeli Analysis vezethet a pontosítás elvetése intézkedéseket, amelyek nem releváns, így már pontosabban az adatokat, amelyekre szüksége van a fókusz, ugyanakkor minimalizálja a háttérzaj.

## <a name="sources-of-monitoring-and-diagnostic-data"></a>Megfigyelési és diagnosztikai adatforrások
Az információk a megfigyelési folyamat által használt származhatnak forrásokból, 1. ábrán látható módon. Az alkalmazás szintjén információk szóló a kódot a rendszer a nyomkövetési naplók származik. A fejlesztők kódjukat keresztül áramló nyomon követése a szabványos megközelítést kell követnie. Például egy metódus bejegyzést el tudná küldeni egy nyomkövetési üzenet, amely megadja a metódus, a jelenlegi időpontnál, mindegyik paraméterhez, és minden egyéb kapcsolódó információt értékének nevét. A be- és kilépés idő rögzítése akkor is lehet hasznos.

Jelentkezzen be a kivételeket, és a figyelmeztetéseket, és győződjön meg arról, hogy megőrizze-e a beágyazott kivételek és figyelmeztetések teljes nyomkövetési. Ideális esetben a tevékenység korrelációs adatokkal (a kérelmek nyomon követése, mert haladnak át a rendszer) együtt, a kódot futtató felhasználó azonosító információkat is rögzíti. És próbálja meg elérni az összes erőforrást, például az üzenetsorok, adatbázisok, fájlok és más függő szolgáltatások naplózni kell. Ez az információ a szoftverhasználat-mérő és naplózási célokra használható.

Számos alkalmazás ellenőrizze a kódtárai és keretrendszerei segítségével végrehajthat olyan gyakori feladatokat, például egy-adattároló elérése vagy a hálózaton keresztül kommunikál. Előfordulhat, hogy ezen keretrendszerek konfigurálható a saját nyomkövetési üzenetek és a nyers diagnosztikai információkat, például a tranzakciós díjakat biztosítanak és adatok továbbítása sikeres és sikertelen műveletek.

> [!NOTE]
> Számos modern keretrendszerek automatikus teljesítmény és a nyomkövetési események közzététele. Ezek az információk rögzítése egyszerűen és tároljuk, ahol feldolgozásra és elemzése segítségével biztosít.
> 
> 

Az operációs rendszer, ahol az alkalmazás futása lehet egy rendszerszintű alacsony szintű információkat, például az i/o-sebességét, a memóriahasználat és a CPU-használat jelző teljesítményszámlálók forrását. Operációs rendszer hibái (például a meg nem nyílik meg megfelelően fájl) is lehet, hogy jelenteni.

Is figyelembe kell venni az alapul szolgáló infrastruktúra és az összetevők, amelyeken a rendszer futtatja. Virtuális gépek, virtuális hálózatok és tárolási szolgáltatások lehetnek források fontos infrastruktúra szintű a teljesítményszámlálók és más diagnosztikai adatokat.

Ha az alkalmazás más külső szolgáltatások, például a webkiszolgáló vagy az adatbázis-kezelő rendszer, ezeket a szolgáltatásokat saját nyomkövetési adatokat, a naplókat, és a teljesítményszámlálók előfordulhat, hogy tegye közzé. Ilyen például az SQL Server dinamikus felügyeleti nézetek nyomon követési műveleteket hajt végre egy SQL Server-adatbázist, és IIS nyomkövetési napló rögzítéséhez a webkiszolgálónak intézett kérelmeket.

A rendszer összetevőinek módosítva lett, és új verziók telepítve vannak, fontos attribútum problémák, események és metrikák minden verzióra. Ezt az információt kell kötni vissza a kiadási folyamatot, hogy egy összetevő egy adott verziójához problémákat gyorsan követni, és javítása.

A rendszer biztonsági problémák bármikor fordulhatnak elő. Például egy felhasználó megkísérelhetik jelentkezzen be egy érvénytelen felhasználói azonosító vagy jelszó. Hitelesített felhasználó megpróbálja erőforrás jogosulatlan hozzáférést szerezni. Vagy a felhasználó előfordulhat, hogy adjon meg egy érvénytelen vagy elavult kulccsal a titkosított adatok eléréséhez. Mindig legyenek naplózva a sikeres és a meghibásodott kéréseket, biztonsággal kapcsolatos információkat.

A szakasz [alkalmazás tagolása](#instrumenting-an-application) további útmutatást nyújt az irányítsa információkat. De stratégiák számos segítségével gyűjtheti össze ezeket az információkat:

* **A rendszer vagy alkalmazás figyelési**. Ezt a stratégiát belső források az alkalmazás, az alkalmazás-keretrendszerek számára, az operációs rendszer és az infrastruktúra belül használja. Az alkalmazás kódjának hozhat létre a saját figyelési adatok figyelmet a jelentősebb pontokat az egyik ügyfélkérelemben-életciklus során. Az alkalmazás nyomkövetési utasítások, előfordulhat, hogy szelektív engedélyezhetők és letilthatók, körülmények tartalmazhatnak. Is esetleg diagnosztika dinamikusan szúrjon egy diagnosztikai keretrendszer használatával. Ezen keretrendszerek általában adja meg a beépülő modulok nem lehet a kódban a különböző instrumentation pontok csatolja, és ezeket a pontokat nyomkövetési adatok rögzítéséhez.
  
    Emellett a kód és/vagy az alkalmazás mögötti infrastruktúra előfordulhat, hogy indíthat eseményeket a kritikus pontokon. Figyelő, amely a ezeket az eseményeket figyelő ügynökök rögzítheti az eseményadatok.
* **Felhasználó figyelési**. Ez a megközelítés rögzíti a felhasználó és az alkalmazás közötti interakció, és minden kérés és válasz áramló betartja. Ez az információ rendelkezhet egy kettős célja: használat mérési az összes felhasználó, és annak meghatározásához, hogy a felhasználók kapnak egy megfelelő szolgáltatásminőség (például a gyors válaszidők, az alacsony késleltetés és a minimális hibák) használható. A rögzített adatok segítségével azonosíthatja a problémás területeket. Ha hiba történik a leggyakrabban. Is használhatja az adatok azonosítására elemek ahol a rendszer lelassul, esetleg csatlakozási pontokhoz, az alkalmazás vagy a szűk keresztmetszetek más fürtözéstípussal miatt. Gondosan alkalmazza ezt a módszert használja, ha esetleg a felhasználói forgalom Hibakeresés és tesztelési célú alkalmazáson keresztül helyreállítására.
  
  > [!IMPORTANT]
  > Érdemes lehet az adatok rögzített valós figyelés kell szigorúan bizalmas, mert a bizalmas adatokat tartalmazhat. Ha a rögzített adatok menti, tárolja biztonságos helyen. Ha szeretné használni az adatok a teljesítmény figyelése, vagy a hibakeresési célra, sáv minden személyes azonosításra alkalmas adatok először.
  > 
  > 
* **Szintetikus felhasználói figyelése**. Ez a megközelítés a saját teszt ügyfél, amely a felhasználó szimulál és a hajtja végre műveletek konfigurálható, de általában több írható. A teljesítmény a teszt ügyfél annak meghatározásához, a rendszer állapotának nyomon követheti. Hogyan reagál a rendszer a magas terhelés alatt, és milyen rendezési kimeneti figyelést akkor jön létre, ilyen körülmények létrehozásához egy terhelés-tesztelési művelet részeként is használhatók a teszt ügyfél több példányát.
  
  > [!NOTE]
  > A figyelés valós és szintetikus felhasználói beleértve, követi, és időpontokat metódushívások és más kritikus kérelem végrehajtása is létrehozható.
  > 
  > 
* **Profilkészítési**. Ez a megközelítés elsősorban azoknak a figyelési és az alkalmazások teljesítményének javítása. Ahelyett, hogy valós és szintetikus felhasználói figyelési működési szinten működik, az alkalmazás fut alacsonyabb szintű információkat rögzíti. Profilkészítési rendszeres mintavételi a végrehajtási állapot egy alkalmazás (mely kódrészletek, amelyek az alkalmazás fut egy időben meghatározására) használatával is létrehozható. Is használhatja a instrumentation mintavételt szúr be a kódot fontos junctures (például a kezdő és záró metódushívások), és milyen módszerek indították, milyen időpontban, és mennyi ideig tartott minden hívás rögzíti. Ezek az adatok meghatározásához az alkalmazás mely részei teljesítményproblémákat okozhat majd elemezheti.
* **Végpontmonitoring kijelző**. Ez a módszer egy vagy több diagnosztikai végpontot, amely kifejezetten a figyelés bekapcsolható az mutatja az alkalmazás használja. A végpont végzett információküldéshez biztosít az alkalmazás kódjának a, és visszatérhet a rendszer állapotával kapcsolatos információkat. Különböző végpontok különböző szempontjairól a funkciók összpontosíthat. A saját diagnosztika ügyfél által küldött kérések végpontokkal való írása, és a válaszok beolvaszt. További információkért lásd: a [állapotfigyelő végpont figyelési mintát](../patterns/health-endpoint-monitoring.md).

Maximális érvényességének ezek a technológiák kombinációját kell használnia.

<a name="instrumenting-an-application"></a>

## <a name="instrumenting-an-application"></a>Egy alkalmazás tagolása
Instrumentation egy a megfigyelési folyamat kritikus részét képezi. Döntéseket lehet jelentéssel bíró teljesítményéről és a rendszer állapotát, csak akkor, ha először rögzíteni az adatokat, amely lehetővé teszi, hogy ezek a döntések. Gyűjtse össze a instrumentation használatával információkat elegendő ahhoz, hogy a teljesítmény értékeléséhez, problémák diagnosztizálásához és anélkül, hogy jelentkezzen be egy távoli az üzemi kiszolgáló nyomkövetési (és a hibakereséshez) végrehajtásához döntéseket kell manuálisan. WMI-adatok általában a metrikák és a nyomkövetési naplók írt információkat tartalmazza.

A nyomkövetési napló tartalmát lehet az alkalmazás által írt adatok szöveges vagy bináris adatok jön létre egy nyomkövetési esemény eredményét (ha az alkalmazás által használt Windows esemény-nyomkövetése--ETW) eredményét. Akkor is létrehozhatók, a rendszer naplóit, amely az infrastruktúra, például webkiszolgálót részeibe származó események. Szöveges napló üzenetek gyakran kell emberek számára olvasható, de a azok is, amely lehetővé teszi az automatikus rendszer könnyen értelmezhető őket formátumban kell írni.

Naplók kategorizálja meg. Nem minden nyomkövetési adatok írása az egyetlen naplót, de külön naplók segítségével jegyezze fel a rendszer különböző működési szempontjait nyomkövetési kimenetét. Majd gyorsan naplóüzenetek szerint szűrheti a megfelelő naplót olvasásakor ahelyett, hogy egyetlen hosszadalmas fájlba feldolgozásához. Soha nem írási információt, amely különböző biztonsági követelményeket (például naplózási információk és adatok hibakeresés) rendelkezik, a naplóhoz.

> [!NOTE]
> A napló is elegendő lehet a fájlrendszerben fájlként, vagy előfordulhat, hogy tárolható néhány más formátumban, például blob Storage tárolóban lévő blob. Naplóadatok is tárolható előfordulhat, hogy több strukturált tárhelyen, például egy tábla sorainak.
> 
> 

Metrikák általában lesz mérték vagy száma néhány szempontja vagy az erőforrás a rendszerben meghatározott időben, egy vagy több hozzárendelt címkék vagy a dimenziók (más néven a *minta*). Egyetlen példány futhat egy metrika nem általában akkor hasznos, elkülönítve. Ehelyett a metrikák van időbeli rögzíthetők. Figyelembe kell venni a kulccsal kapcsolatos probléma, mely metrikák rögzíteni kell, és hogy milyen gyakran. Adatok metrikáihoz túl gyakran létrehozása adhat a rendszer jelentős további terhelése, mivel rögzítésével metrikák ritkán okozza, amelyeken a esetekben jelentős esemény történt, hogy vezethet. A szempontok metrika metrika függ. Például egy kiszolgálón CPU-felhasználás előfordulhat, hogy jelentős eltérések lehetnek a másodikból másik, de magas kihasználtsága válik a problémát, csak ha hosszú élettartamú a perc alatt.

<a name="information-for-correlating-data"></a>

### <a name="information-for-correlating-data"></a>Információk adatok használatával történik
Könnyen egyes rendszerszintű teljesítményszámlálók figyelése, metrikák erőforrások rögzítése és alkalmazás nyomkövetési adatok lekérését a különböző naplófájlokat. De bizonyos űrlapok figyelési igényelnek a figyelési folyamat összefüggéseket a több forrásból beolvasott adatok elemzése és diagnosztikai szakasza. Ezeket az adatokat több űrlap tarthat a nyers adatok, és a folyamat biztosítani kell tennie képezi le különböző űrlapok megfelelő WMI-adatok. Például az alkalmazás-keretrendszer szintjén feladat lehet, hogy azonosítható, ha egy szálat. Az alkalmazáson belül ugyanaz a munkahelyi hozzárendelve a felhasználóhoz, aki ezt a feladatot hajt végre a felhasználói azonosító lehet.

Is hogy nem valószínű, hogy lehet, hogy egy 1:1 leképezési szálak és a felhasználói kérelmek közötti aszinkron műveletek előfordulhat, hogy ismét felhasználni, az azonos szálak egynél több felhasználó nevében műveletek végrehajtásához. A fontos információk nehezíti további, egyetlen kérelem előfordulhat, hogy kell kezelnie egynél több szál rendszeren keresztül végrehajtási flow-ként. Ha lehetséges egyes kérelmek társítani egy egyedi tevékenység azonosítója, amely segítségével a rendszer a kérés környezete részeként propagálja. (A technika, és a nyomkövetési adatokat is beleértve tevékenység azonosítók függ a nyomkövetési adatok rögzítésére szolgáló technológia.)

Összes figyelési adatot időbélyeggel ellátott ugyanúgy kell lennie. Konzisztencia jegyezze fel a összes dátum és idő egyezményes világidő használatával. Ez segítséget nyújt könnyebben nyomkövetési események sorozatát.

> [!NOTE]
> Előfordulhat, hogy a különböző időzónák és hálózatok működő számítógépek nem lesznek szinkronizálva. Nem attól függ, hogy kizárólag időbélyegeket használja több számítógép is WMI-adatok használatával történik.
> 
> 

### <a name="information-to-include-in-the-instrumentation-data"></a>A WMI-adatok foglalandó adatok
Amikor eldönti, hogy mely WMI-adatok gyűjtésére, vegye figyelembe a következő szempontokat:

* Ellenőrizze, hogy a nyomkövetési események által rögzített adatokat gép és a HR-részleg olvasható. Ezeket az adatokat, lehetővé teszi a naplóadatok automatizált feldolgozási rendszerek között, és a műveletek és a naplók beolvasásakor személyzet mérnöki konzisztencia biztosításához jól meghatározott sémák fogad el. Környezeti adatokat tartalmaznak, például a telepítési környezet, a gép, amelyen a folyamat fut, a folyamat, és a hívási verem részleteit.  
* Adatgyűjtés csak szükség esetén, mert akkor adhat meg a rendszer jelentős terhelésének engedélyezése. Profilkészítési instrumentation használatával rögzíti (például egy metódushívás) esemény minden alkalommal következik be, mivel mintavételi rekordok csak a kijelölt eseményeket. A kijelölés időalapú lehet (miután minden  *n*  másodperc), vagy gyakoriság-alapú (után minden  *n*  kérelmek). Események nagyon gyakran fordul elő, ha által instrumentation profilkészítési előfordulhat, hogy túl sok terhet és okozhat maga általános teljesítményét. Ebben az esetben a mintavétel módszer előnyösebb lehet. Azonban ha az események gyakorisága alacsony, mintavételi előfordulhat, hogy teljesíti az őket. Ebben az esetben a instrumentation jobb megközelítés lehet.
* Adjon meg egy fejlesztői vagy a rendszergazda az összes kérelem forrásának megfelelő környezet. Ebbe beletartozhatnak valamilyen tevékenység azonosítója, amely azonosítja a kérelmet egy adott példányához. Ezt a tevékenységet a számítási végzett munka, és a használt erőforrások összefüggéseket használható információkat is tartalmazhat. Vegye figyelembe, hogy előfordulhat, hogy ez a munkahelyi kereszt-folyamat, illetve a számítógép határokat. Mérés, a környezet is tartalmaznia kell (közvetlenül vagy közvetve keresztül más korrelált információk) ügyfél számára, hogy a kérelem elvégzendő mutató hivatkozás. Ebben a kontextusban az alkalmazás állapotára vonatkozó figyelési adatokat rögzítésének időben értékes információkat biztosít.
* Rögzítse az összes kérelmet, és a helyek vagy régiókban, amelyből a kérések száma. Ez az információ segít a meghatározása, hogy vannak-e bármilyen helyspecifikus csatlakozási pontokhoz. Ez az információ is hasznos lehet meghatározni, hogy e particionálni egy alkalmazás vagy az azt használó adatokat.
* Jegyezze fel, és gondosan rögzítése a kivételek részletei. Gyakran kritikus hibakeresési információ elvész gyenge kivétel miatt. A kivételek fellépése, amelyek az alkalmazás jelez, beleértve a belső kivételek és más környezeti teljes körű információkat rögzíti. Ha lehetséges közé tartozik a hívási verem.
* Lehet, az adatok, amelyek mérik a különböző elemekhez, az alkalmazás, egységes Ez segít a események elemzésével, és azokat a felhasználói kérelmek adatok. Fontolja meg egy átfogó és konfigurálható naplózása csomag használata összegyűjteni ahelyett, hogy a fejlesztők számára, hogy elfogadják a megközelítési valósítják meg a rendszer különböző részei függően. Adatokat gyűjt a kulcsfontosságú teljesítményszámlálókról, például a kötet i/o végrehajtás alatt álló hálózathasználat, kéréseket, a memória használata, és a CPU-felhasználás száma. Néhány infrastruktúra-szolgáltatásokat nyújthatnak a saját specifikus számlálókat, például az adatbázis, a sebesség, amellyel tranzakciók történik, és sikeres vagy sikertelen tranzakciók száma kapcsolatok száma. Alkalmazások is definiálhat a saját egyes teljesítményszámlálókat.
* Külső szolgáltatások, például az adatbázis-rendszerek, webszolgáltatások vagy egyéb rendszerszintű szolgáltatások, az infrastruktúra részét képező összes hívások naplózása. Jegyezze fel minden hívás végrehajtásához szükséges idő kapcsolatos információkat és a sikeres, vagy a a hívás sikertelen. Ha lehetséges az összes alkalmazás adatai rögzítési újrapróbálkozások és bármely átmeneti hibák esetén fellépő hibák.

### <a name="ensuring-compatibility-with-telemetry-systems"></a>Telemetria rendszerekkel való kompatibilitás biztosítása
Sok esetben az információt, amely instrumentation generált események sorozataként és elemzések egy külön telemetriai rendszerre átadva. A telemetriai adatok rendszer általában független bármely adott alkalmazás vagy topológia, de azt várja, amely általában egy séma határozza meg az adott formátumú adatokat. A hatékony sémában, amely meghatározza az adatok és a típusokat, amelyek a telemetriai adatok rendszer fogadására képes szerződés. A séma platformokon és eszközökön számos érkező adatok lehetővé kell általánosítva.

A közös séma tartalmaznia kell mezőket vonatkozó összes instrumentation esemény, például az esemény nevét, az esemény időpontja, a küldő és a részletek szükséges adatok (például a felhasználói azonosító Eszközazonosítót az eseményeket az IP-címe és az Alkalmazásazonosító). Ne feledje, hogy lévő eszközök események, előfordulhat, hogy ablakába, ezért a séma nem kell az eszköz-típustól függnek. Emellett a különböző eszközök előfordulhat, hogy előléptetése események ugyanahhoz az alkalmazáshoz; az alkalmazása által támogatott központi vagy egyéb formája kereszt-eszköz.

A séma, amely kapcsolódik a különböző alkalmazások által közösen használt adott célra tartomány mezők is tartalmazhatja. Ez az információ a kivételek, alkalmazás kezdő és záró eseményeket, és a sikeres és/vagy webszolgáltatás API-hívás sikertelen lehet. Minden alkalmazás, amely ugyanazokat a tartomány mezők használni kell hozható létre olyan események, közös jelentések és elemzés kialakítani engedélyezése.

Végül a séma tartalmazhat egyéni mezők rögzítésének alkalmazásspecifikus események részletes adatait.

### <a name="best-practices-for-instrumenting-applications"></a>Alkalmazások tagolása ajánlott eljárásai
Az alábbi lista a felhőben futó elosztott alkalmazások tagolása ajánlott eljárásai foglalja össze.

* Ne naplók könnyen olvasható és könnyen értelmezhető. Használjon strukturált naplózás, ahol csak lehetséges. Rövid és informatív üzeneteket a lehet.
* Összes napló a forrás azonosítása, és adja meg és az információk, mivel a rendszer minden naplóbejegyzés ír.
* Időzóna és a formátum használata minden időbélyegeket. Ez segít a hardver és a különböző földrajzi régióban futó szolgáltatások műveletek események összefüggéseket.
* A megfelelő naplófájlba írásakor és kategorizálása a naplók.
* Nem fedjük fel a rendszer bizalmas adatait vagy a felhasználók személyes információt. Megtisztítás ezt az információt, mielőtt jelentkezett, de győződjön meg arról, hogy megőrzi a vonatkozó adatokat. Például az azonosító és jelszó eltávolítása minden adatbázis-kapcsolati karakterláncok, de a naplóba bejegyezni további adatait, hogy valamelyik elemzőnek megállapíthatja, hogy a rendszer fér hozzá a megfelelő adatbázishoz. Minden kritikus kivételeket, de a rendszergazdája számára, kapcsolja be a naplózást be- és kikapcsolását kivételeket és figyelmeztetések alacsonyabb szinteken. Emellett rögzítheti és újrapróbálkozási logika kapcsolatos összes információ. Ezeket az adatokat a rendszer átmeneti állapotának figyelése a hasznos lehet.
* Nyomkövetés kívüli folyamat indított, például külső webszolgáltatások és adatbázisok kérelmek.
* Naplóüzenetek ne keverje különböző biztonsági követelmények a azonos naplófájlban. Például nem írható hibakeresési és naplózási azonos naplózandó információk.
* Kivételével kapcsolatos naplózási eseményeket, ellenőrizze, hogy az összes naplózási hívás tűz-és-elfelejti műveleteket, amelyek nem blokkolják az üzleti tevékenységre előrehaladását. Naplózási eseményeket olyan kivételes, mivel az üzleti szempontból kulcsfontosságúként, illetve az üzleti tevékenységre alapvető részeként besorolhatók.
* Győződjön meg arról, hogy naplózási bővíthető, és nem rendelkezik konkrét, meghatározott cél bármely közvetlen függőség tartozhat. Például ahelyett, hogy adatok írásakor használatával *System.Diagnostics.Trace*, absztrakt illesztőfelület határozza meg (például *ILogger*), amely elérhetővé teszi a naplózási módszerek és, amelyek segítségével kell végrehajtani megfelelő módon.
* Győződjön meg arról, hogy az összes naplózási hibamentes-e, és soha nem elindítja az egymásra épülő hibáit. Naplózás nem kell throw kivételek.
* Folyamatban lévő iteratív folyamat instrumentation tekinti, és nem csak akkor, ha a probléma rendszeresen, ellenőrizze a naplókat.

## <a name="collecting-and-storing-data"></a>Összegyűjtése és adatok tárolása
A megfigyelési folyamat gyűjtemény szakasza foglalkozik, amely instrumentation hoz létre, a könnyebb felhasználásához az elemzés/elemzés céljából. szakaszra vonatkozóan, és az átalakított adatok mentése megbízható tárolt adatok formázása adatainak lekérése során. A WMI-adatok, amelyek gyűjtse össze egy elosztott rendszer különböző részei a különböző helyek és a különböző formátumokban is tárolható. Például az alkalmazás kódjában előfordulhat, hogy nyomkövetési naplófájlok készítése és készítése alkalmazás eseménynapló-adatokat, mivel más technológiák útján rögzíthetők teljesítményszámlálót mutat be, figyelheti a kulcsfontosságú elemeit annak az infrastruktúra, amely az alkalmazás használja. Bármely külső összetevőkkel és szolgáltatásokkal, hogy az alkalmazás által használt nyújthatnak rendszerállapottal kapcsolatos információkat a különböző formátumokban külön nyomkövetési fájlok, a blob-tároló, vagy még akkor is egyéni adattárat.

Adatgyűjtés gyakran önállóan futtatható az alkalmazás, amely hoz létre a WMI-adatok gyűjtése szolgáltatáson keresztül történik. 2. ábra ebbe az architektúrába, a rendszerállapot-adatgyűjtés alrendszer kiemelés mutatja be.

![Példa WMI-adatok gyűjtése](./images/monitoring/TelemetryService.png)

*2. ábra. WMI-adatok gyűjtése*

Vegye figyelembe, hogy ez egy egyszerűsített nézete. A szolgáltatás nem feltétlenül egyetlen folyamat, és előfordulhat, hogy a különböző gépeken futó sok része alkotják, a következő szakaszokban ismertetett módon. Továbbá ha néhány telemetriai adatok elemzését gyorsan kell elvégezni (közbeni elemzés, a szakaszban leírt módon [támogató működés közbeni, gyakran és ritkán használt analysis](#supporting-hot-warm-and-cold-analysis) a jelen dokumentum későbbi), helyi összetevők működését a szolgáltatás előfordulhat, hogy feladatokat elemzés azonnal. 2. ábra az ebben a helyzetben a kijelölt eseményeket ábrázol. Analitikus feldolgozás után, az eredmények küldhető közvetlenül a képi megjelenítés és a riasztási alrendszer. Van kitéve a meleg vagy a cold elemzési adatokat van tárolt várja feldolgozása közben.

Az Azure-alkalmazások és szolgáltatások, Azure Diagnostics egy lehetséges megoldást nyújt adatok rögzítéséhez. Az Azure Diagnostics adatait gyűjti össze az egyes számítási csomópontok az alábbi forrásokból összesíti az és majd feltölti az Azure Storage:

* IIS-naplók
* Az IIS Kérelemszűrése sikertelen naplók
* Windows-Eseménynapló
* Teljesítményszámlálók
* Összeomlási memóriaképek
* Az Azure Diagnostics infrastruktúra naplók  
* Egyéni hibanaplókat
* .NET EventSource
* Jegyzékfájl-alapú ETW

További információkért lásd: a cikk [Azure: Telemetriai alapjait és hibaelhárítás](http://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx).

### <a name="strategies-for-collecting-instrumentation-data"></a>Stratégiák a WMI-adatok gyűjtése
A rugalmas jellegét a felhőben, és ne legyen szükség a manuálisan lekérése telemetriai adatokat minden csomópont a rendszerben, figyelembe véve gondoskodik az adatok átvitele egy központi helyen, és konszolidált. A rendszer több adatközpontot is lehet hasznos, ha először begyűjtése, összevonása akkor történjen meg, és adatok régiónként-alapon tárolja, és majd összesítése a regionális adatok egyetlen központi rendszerben szeretné.

Sávszélesség-optimalizálásához választhatja adattömböket, mint a kötegek üzenetfejléceket adatok átvitele. Azonban az adatokat kell nem később határozatlan ideig, különösen akkor, ha időérzékeny információkat tartalmaz.

#### <a name="pulling-and-pushing-instrumentation-data"></a>*Húzza és WMI-adatok küldését*
A rendszerállapot-adatgyűjtés alrendszer aktívan beolvasható WMI-adatok a különböző naplókat, valamint a más forrásokból, az alkalmazás egyes példányainak (a *lekéréses modell*). Vagy a passzív fogadó, amely megvárja-e az adatok küldését a az alkalmazás minden példánya alkotó összetevők működhet (a *leküldési modelljével*).

A lekéréses modell végrehajtási egyik módszer, hogy figyelőügynökök futtató helyi az alkalmazás minden példányát használja. Figyelési ügynök külön folyamat, amely rendszeres időközönként beolvassa (ponttá) telemetriai adatokat a helyi csomópont be, és írja közvetlenül ezeket az információkat, amelyek az alkalmazás összes példányát központi tárolása. Azt a mechanizmust, amely megvalósítja az Azure Diagnostics. Egy Azure webes vagy feldolgozói szerepkör minden egyes példánya beállítható úgy, hogy a rögzítési diagnosztikai és egyéb nyomkövetési adatok helyben tárolt. A figyelési ügynök, amely minden példány együtt fut az Azure Storage másolja át a megadott adatokat. A cikk [diagnosztika engedélyezésével az Azure Cloud Services és a virtuális gépek](/azure/cloud-services/cloud-services-dotnet-diagnostics) részletesen ismerteti a folyamatot. Bizonyos elemek, például az IIS naplóit, összeomlási memóriaképek és egyéni hibanaplókat, a blob-tároló kerülnek. A table storage a Windows Eseménynapló ETW-események és teljesítményszámlálók adatait rögzíti. 3. ábra szemlélteti a mechanizmus.

![A figyelési ügynök használatával információkat le, és írja a megosztott tárolóba ábrája](./images/monitoring/PullModel.png)

*3. ábra. A figyelési ügynök használatával információkat le, és írja a megosztott tárolóba*

> [!NOTE]
> A figyelési ügynök használatával kiválóan alkalmas a WMI-adatok természetes lekért adatforrás rögzítése. Egy példa az SQL Server dinamikus felügyeleti nézetek vagy egy Azure Service Bus-várólista hossza információt.
> 
> 

Legyen megvalósítható a csak korlátozott számú csomópontok egyetlen helyen futó csak kevés számítógépet érintő alkalmazások telemetriai adatainak tárolásához leírt módon. Azonban egy összetett, skálázható, globális felhő alkalmazás hatalmas mennyiségű adatot előfordulhat, hogy generálása több száz webes és feldolgozói szerepkörök, adatbázis szilánkok és egyéb szolgáltatásokat. Az adatok foglalkozók is könnyen ne terhelje, tovább a i/o-sávszélesség egyetlen központi helyen érhető el. Ezért a telemetriai adatok megoldásának méretezhető, ezáltal megakadályozhatja, hogy a rendszer kiterjeszti a szűk forráscsoportként kell. Ideális esetben a megoldás (például naplózási vagy számlázási adatok) fontos figyelési adatok elvesztése, ha a rendszer nem sikerül a kockázatok csökkentése redundancia fokú kell tartalmaznia.

A következő problémák is létrehozható queuing, a 4. ábrán látható módon. Az ebbe az architektúrába, a helyi figyelési ügynök (ha megfelelően konfigurálható) vagy egyéni adatgyűjtés-szolgáltatás (Ha nem) POST-adatokat annak a várólistára. (A tárolás szolgáltatás írása a 4. ábra) aszinkron módon fut külön folyamatban lekéri az adatokat a várólistán szereplő, és írja a megosztott tárhelyhez. Az üzenet-várólista nem megfelelő-e ebben a forgatókönyvben, mert a "legalább egyszeri" biztosít, amelyek segítenek, győződjön meg arról, hogy várólistán lévő adatok nem vesznek után feladás szemantikáját. A storage szolgáltatás írása egy külön feldolgozói szerepkör használatával is létrehozható.

![A várólisták puffer WMI-adatok használatával ábrája](./images/monitoring/BufferedQueue.png)

*4. ábra. A várólisták puffer WMI-adatok használatával*

A helyi adatgyűjtés szolgáltatás adatokat adhat egy várólista átvétel után azonnal. A várólista pufferként, és a tárolás szolgáltatás írása beolvashatja és a saját tempójában írni. Alapértelmezés szerint egy várólista érkezési sorrendben történő kiküldési szinten működik. De rangsorolhatja üzenetek keresztül a várólista gyorsító őket, ha gyorsabban kell kezelni álló adatokat tartalmaznak. További információkért lásd: a [prioritású várólistára](https://msdn.microsoft.com/library/dn589794.aspx) mintát. A különböző csatornákon (például a Service Bus-üzenettémakörök) segítségével azt is megteheti, attól függően, hogy az űrlap elemzésfeldolgozási, amely szükséges a különböző helyekre adatok közvetlen.

A méretezhetőség a tárolás szolgáltatás írása több példányát is futtathatja. Ha nagy mennyiségű esemény, használhatja az eseményközpontok a különböző számítási erőforrásokat, az adatfeldolgozás és -tárolás adatokat átirányítani.

<a name="consolidating-instrumentation-data"></a>

#### <a name="consolidating-instrumentation-data"></a>*Összevonja a WMI-adatok*
A WMI-adatok az adatgyűjtés szolgáltatás egy alkalmazás egyetlen példányát kiolvassa lehetőséget nyújt az honosított rálátást enged és annak a példánynak teljesítményét. A rendszer általános állapotának felmérésére fontos vonják össze a helyi nézetében lévő adatok az egyes funkcióit. Ezt követően az adatokat a tárolja, de egyes esetekben is érhet el, az adatgyűjtés a következők szerint hajthat végre. Ahelyett, hogy közvetlenül a megosztott tárolóba írása, a WMI-adatok egy különálló összevonási szolgáltatás, amely egyesíti az adatokat, és úgy működik, mint egy szűrő és törlési folyamat is továbbítása. Például a azonos korrelációs adatok, például Tevékenységazonosítót WMI-adatok által is. (Ez akkor lehet, hogy egy felhasználó elindítja az egyik csomópont üzleti műveletet, és ezután lekérdezi egy másik csomópontjára továbbítja csomópont meghibásodik, vagy attól függően, hogy a terheléselosztás konfigurálását.) Ez a folyamat is észleli, és távolítsa el az ismétlődő adatokat (mindig annak a lehetősége, ha a telemetriai adatok szolgáltatás használ a leküldéses WMI-adatok kimenő Storage üzenetsorokat). 5. ábra mutatja be: Ez a struktúra.

![A szolgáltatás használatával vonják össze a WMI-adatok – példa](./images/monitoring/Consolidation.png)

*5. ábra. Külön szolgáltatással összefogása és WMI-adatok törlése*

### <a name="storing-instrumentation-data"></a>WMI-adatok tárolása
Az előző beszélgetéseket, amely WMI-adatok tárolja módja ahelyett, hogy simplistic nézetét rendelkezik ábrázolva. A valóságban azt is célszerű a különböző típusú adatok tárolására, amelyben minden várhatóan használt módszer a legmegfelelőbb technológiák használatával.

Például az Azure-blob és table storage rendelkezik néhány Hasonlóságok elért most módon. Azonban lehetnek korlátozások a hajthat végre őket a műveletekben, és az adatokat azok vonatkozik, amelyek granularitása Igen különbözőek. Ha további analitikai műveleteket végezhet, vagy az adatok teljes szöveges keresésre van szüksége, több megfelelő optimalizált képességeket biztosító lekérdezések és adatelérési bizonyos típusú adatok tárolási használandó lehet. Példa:

* Teljesítmény számlálóadatok alkalmi elemzés engedélyezése SQL-adatbázisban tárolható.
* Nyomkövetési naplók jobban tárolódhat Azure Cosmos DB.
* Biztonsági adatokat HDFS csak írható.
* Teljes szöveges keresés által kért információ (amely a gazdag indexelési keresések is sebessége) Elasticsearch keresztül is tárolhatók.

Egy másik szolgáltatást, amely rendszeres időközönként lekéri az adatokat a megosztott tároló, a partíciók és a szűrők célja az adatok, és ezután ír egy megfelelő összességének adattárolókhoz 6. ábrán látható módon valósíthatja meg. Egy másik módszert is, hogy tartalmazza ezt a szolgáltatást a konszolidáció és törlési folyamat és az adatok írása közvetlenül ezekkel az áruházakkal mivel azt rendelkezik beolvasott ahelyett, a közbenső mentése megosztott tárterület. Minden módszer van annak előnyeit és hátrányait. Egy külön particionálási szolgáltatás üzembe csökkenti annak az Összevonás és karbantartása szolgáltatás terhelését, és lehetővé teszi, hogy legalább (attól függően, hogy mennyi adatot őrzi meg a megosztott tároló) szükség esetén újra létrejön a particionált adatok egy részét. Azonban ez további-erőforrásokat használ fel. Is valószínűleg késleltetés az alkalmazás feltünteti a WMI-adatok fogadását, és ezeket az adatokat átváltását végrehajthatóként információk között.

![Particionálás és az adatok tárolása](./images/monitoring/DataStorage.png)

*6. ábra. Particionálás analitikai adatok és a tárhellyel kapcsolatos követelmények*

Az azonos WMI-adatok több célra lehet szükség. Például teljesítményszámlálók segítségével adja meg a rendszer teljesítményét az idő múlásával ügyfélállapotainak. Ezeket az információkat az ügyfél számlázási generálhassák egyéb használati adatok lehet kombinálni. Ezekben a helyzetekben ugyanazokat az adatokat több célra, például egy dokumentum-adatbázis, amely működhet, és a hosszú távú tárolási a számlázási adatokat tároló és a többdimenziós áruházból kezelése bonyolult teljesítmény analytics lehet küldeni.

Azt is figyelembe kell venni, hogyan sürgősen a adatokra szükség. Bemutatja, hogy a riasztási adatok használatával kell elérni gyorsan, ezért érdemes lehet tárolt adatok gyors és indexelt vagy felépítve, hogy a riasztási rendszer hajt végre a lekérdezések optimalizálását. Egyes esetekben szükség lehet a telemetriai adatok szolgáltatás, amely összegyűjti a minden egyes csomóponton formázása és a helyileg menteni az adatokat, hogy a riasztási rendszer helyi példányának gyorsan értesítheti, az esetleges problémákat. A storage szolgáltatás az előző ábrán is látható, és központilag tárolja, ha más célokra is szükséges írása ugyanazokat az adatokat képes továbbítani.

További használható információkat figyelembe veendő analysis jelentéskészítéshez, valamint a múltbeli trendek gyorsabban üzenetfejléceket, amely olyan módon, amely támogatja az adatbányászat és az ad hoc lekérdezéseket is tárolhatók. További információkért tekintse meg a szakasz [működés közbeni, gyakran és ritkán használt elemzési támogató](#supporting-hot-warm-and-cold-analysis) újabb ebben a dokumentumban.

#### <a name="log-rotation-and-data-retention"></a>*Naplóváltás és az adatok megőrzése*
Instrumentation hozhat létre a jelentős mennyiségű adatot. Ezeket az adatokat több helyen, a nyers naplófájlok nyomkövetési fájlok kezdve lehessen vonni és egyéb információkat a konszolidált minden csomópont rögzített tisztítani és particionált nézet a tárolt megosztott adatokról. Néhány esetben az adatok feldolgozott és továbbított, után az eredeti forrás nyers adatok távolíthatók el minden csomópont. Más esetben egyszerűen hasznos a nyers adatok mentéséhez vagy szükséges lehet. Hibakeresési célra generált adatok például előfordulhat, hogy legjobban hagyható nyers formájában érhető el, de tudja majd elveti gyorsan hibáit rendelkezik lett javítása után.

Teljesítményadatok gyakran van hosszabb élettartamát, így gyorsabban teljesítménytrendek és a kapacitástervezés is használható. Az összevont nézetet ezen adatok általában tartják online véges időtartamra és a gyors hozzáférés engedélyezéséhez. Ezt követően az archiválható vagy elvetett. Adatgyűjtés-mérés és számlázási ügyfelek előfordulhat, hogy határozatlan ideig menteni kell. Emellett szabályozó igények miatt előfordulhat, hogy naplózási és biztonsági okokból gyűjtött információkat is archivált és menteni kell. Ezeket az adatokat is bizalmas, és előfordulhat, hogy a titkosított vagy más módon védett az illetéktelen kell. A felhasználói jelszavakat és egyéb adatait, amelyik alkalmas lehet véglegesíteni identitás csalás rögzíteni soha nem kell. Ezeket az adatokat az adatokból kell törlődik, mielőtt a rendszer tárolja.

#### <a name="down-sampling"></a>*Lefelé-mintavétel*
Akkor célszerű múltbeli adatok tárolására, hosszú távú trendek ismerhető. Ahelyett, hogy a régi adatok mentése egészében, esetleg le-minta csökkentése a probléma megoldásához és a tárolási költségek csökkentése az adatokat. Tegyük fel helyett mentése perc által percenként teljesítménymutatók, akkor összevonhatja adatokat, amelyek több mint egy hónappal régi ahhoz, hogy az űrlap egy óra órán keresztül nézetben.

### <a name="best-practices-for-collecting-and-storing-logging-information"></a>Ajánlott eljárások az összegyűjtése és naplózási információk tárolása
Az alábbi lista a gyakorlati tanácsok a rögzítése és tárolása naplózási információk foglalja össze:

* A figyelési ügynök vagy adatgyűjtési szolgáltatás folyamaton-szolgáltatásnak futnia kell, és egyszerű telepítendő kell lennie.
* Az összes kimenetét a figyelési ügynök, vagy adatgyűjtés szolgáltatás kell lennie egy független formátum, amely független a gép, az operációs rendszer vagy a hálózati protokoll. Például az adatok egy önálló leíró formátumban, például a JSON-NÁ, MessagePack, vagy Protobuf helyett ETL/ETW létrehozása. Egy szabványos formátumban lehetővé teszi, hogy a rendszer feldolgozási folyamatok; összeállításához olvassa el, -átalakítási és -adatokat küldeni a egyeztetett formában összetevőket könnyen integrálható.
* A figyelés és az adatgyűjtési folyamat hibamentes kell lennie, és nem kell elindítani a kaszkádolt hiba feltételeket.
* Az adatokat küld egy adatokat fogadó átmeneti hiba esetén a figyelési ügynök vagy az adatgyűjtés szolgáltatás kell készíteni, hogy a legújabb információk elküldése először a telemetriai adatok sorrendjének módosításához. (A figyelési ügynök/adatgyűjtési szolgáltatás előfordulhat, hogy dönthetnek úgy, hogy dobja el a régebbi adatokat, vagy később való szinkronizálásához, saját belátása szerint részére és helyileg menteni.)

## <a name="analyzing-data-and-diagnosing-issues"></a>Adatok elemzése és a problémák diagnosztizálása
A figyelés és diagnosztika folyamat fontos részét elemzi az összegyűjtött adatokat a teljes jól lesznek a rendszer kép beszerzése. A saját KPI-ket és a metrikák kell meghatározta, és fontos, hogy az elemző követelményeknek gyűjtött adatok felépítésének ismertetése. Is fontos tudni, hogyan különböző metrikák és napló-fájlt rögzített az adatok visszamenőleges korrelációban állnak, mert ezek az információk kulcs nyomkövetési események sorozatát kell, és a felmerülő problémák diagnosztizálásához.

A szakaszban leírt módon [WMI-adatok konszolidálása](#consolidating-instrumentation-data), a rendszer részét képező összes adatok általában helyileg rögzített, de általában kell használható együtt más helyeken, amely részt vesz a létrehozott adatok a a rendszer. Ezt az információt igényel annak érdekében, hogy pontosan adatok egyesítése gondos korrelációs. A használati adatok művelet például előfordulhat, hogy span a csomópontot, amelyen a webhely, amelyhez a felhasználó csatlakozik, a csomópont egy külön szolgáltatás érhető el ezt a műveletet, és egy másik csomópontjára tárolt adatok tárolási részeként futó. Ezt az információt kell kötni segítségével adja meg a műveletet az erőforrás- és feldolgozási használati átfogó képet. Néhány előzetes feldolgozás és az adatok szűrése akkor fordulhat elő, a csomóponton, amelyen az adatok rögzített, mivel összesítési és formázás fordul központi csomóponton.

<a name="supporting-hot-warm-and-cold-analysis"></a>

### <a name="supporting-hot-warm-and-cold-analysis"></a>Működés közbeni, gyakran és ritkán használt elemzési támogatása
Elemzése és újraformázása a képi megjelenítéshez tartozó adatokat, jelentések és riasztási céljából összetett folyamat lehet, amely a saját erőforrások készletét akkor. Néhány űrlapok figyelési sürgős és hatékony azonnali adatelemzési igényel. Ez más néven *elemzés közbeni*. Például az elemzések, amelyek szükségesek a riasztás és a biztonsági figyelést (például a rendszer a támadás észlelése) egyes funkcióit. Ezekből a célokból szükséges adatok gyors elérhető és a hatékony feldolgozás strukturált kell lennie. Egyes esetekben szükség lehet áthelyezni az elemzés feldolgozását az egyes csomópontokon, az adatok tárolási helye.

Elemzés más formáinak kevesebb idő kritikus, és szükség lehet néhány számítási és összesítési után a nyers adatok érkezett. Ez a lehetőség *elemzés mozog*. Teljesítményelemzés gyakran ebbe a kategóriába tartozik. Ebben az esetben egy elkülönített, egyetlen teljesítményesemény valószínű statisztikailag jelentős. (Ez oka lehet egy hirtelen csúcs vagy probléma merült.) Az események sorozatát adatait a rendszer alapteljesítményét a megbízhatóbb kép kell biztosítania.

Meleg analysis is egészségügyi problémák diagnosztizálásához használható. Olyan állapotesemény általában a gyors elemzés feldolgozása, és azonnal is hoz létre riasztást. Egy olyan operátort kell tudni elemezze mélyebben a állapotesemény okait meleg elérési úton található adatok vizsgálatával. Ezek az adatok tartalmaznia kell a állapotesemény kiváltó vezettek eseményekkel kapcsolatos információk.

Néhány felügyeleti típus hozhat létre a hosszabb távú adatokat. Az elemzés hajtható végre egy későbbi időpontban, valószínűleg egy előre meghatározott ütemezés szerint. Bizonyos esetekben az elemzés előfordulhat, hogy kell elvégeznie a nagy mennyiségű adat egy meghatározott időtartamra vonatkozóan rögzített összetett szűrést. Ez a lehetőség *cold analysis*. A kulcs mérete, hogy az adatok tárolási biztonságosan után már megtörtént. Például használat, figyelés és naplózás szüksége van a rendszer rendszeres pontokon állapotának pontos képet idő, de az állapotadatokat nem feltétlenül szükséges összegyűjtött lett után azonnal feldolgozásra.

Operátor is cold elemzés segítségével adja meg az adatok prediktív állapotelemzésre. Az operátor korábbi összegyűjteni adott időszakon belül, és együtt használja az aktuális egészségügyi adatok (beolvasva a gyakran használt adatok elérési útja) követni a trendeket, amely hamarosan egészségügyi problémákat okozhat. Ezekben az esetekben szükség lehet a riasztást, hogy a javítási műveletek is lehet képernyőfelvételt készíteni.

### <a name="correlating-data"></a>Adatok használatával történik
Instrumentation rögzíti az adatokat a rendszer állapotáról pillanatképet tud biztosítani, de elemzés célja, hogy az adatok hajtható végre. Példa:

* Mi okozta egy adott időpontban a rendszer szintjén betöltése intenzív i/o?
* Ennyi az adatbázis-műveletek nagy számú eredményét?
* Ez az adatbázis válaszidőt, a tranzakciók másodpercenkénti száma, és fordítva válaszidőn alkalmazás, az azonos ettõl a pillanattól?

Ha igen, egy javító műveleteket, lehet, hogy csökkentse a terhelés lehet shard az adatok több kiszolgáló. Ezenkívül kivételek felmerülő bármilyen szinten, a rendszer egy hiba miatt. Egy szint a gyakran kivétel a fenti szinten egy másik hibát váltja ki.

Ezen okok miatt kell tennie a figyelési adatok, a rendszer és a rajta futó alkalmazások állapotát átfogó képet létrehozásához minden szinten különböző típusú összefüggéseket. Ezután ezek az információk segítségével döntéseket hozzanak arról, hogy a rendszer akadálytalanul vagy nem működik, és határozza meg, mi a rendszer minőségének végezhető.

A szakaszban leírt módon [információk használatával történik az adatok](#information-for-correlating-data), győződjön meg arról, hogy a nyers WMI-adatok megfelelő környezet és a tevékenység azonosítója segítő információkat tartalmaz a szükséges összesítések események használatával történik. Ezenkívül ezek az adatok különböző formátumokban lehet tartani, és ezeket az adatokat átalakíthatja elemzéshez szabványos formában elemzése szükség lehet.

### <a name="troubleshooting-and-diagnosing-issues"></a>Hibaelhárítás és a problémák diagnosztizálása
A diagnózisához képes hibáit, vagy váratlan viselkedést, beleértve a legfelső szintű OK elemzések végrehajtását okának megállapításához. Az információk szükséges általában a következők:

* Részletes információk az eseménynaplóban és nyomkövetésben, vagy a teljes rendszerben, vagy egy megadott időszakban egy adott alrendszer.
* Fejezze be a kivételeket és egy meghatározott időtartamon belül a rendszer vagy egy megadott alrendszer előforduló hibák bármely megadott szint eredő híváslánc megjelenik.
* Összeomlási memóriaképek bármely sikertelen folyamatok bárhol a rendszerben vagy egy megadott alrendszer a megadott időszak alatt történjen.
* Tevékenység naplózza a meghatározott időszakon belül minden felhasználó vagy a kijelölt felhasználók elvégzett műveletek rögzítése.

A rendszer-architektúra és a különböző összetevők állítható össze a megoldás részletes műszaki ismeretének gyakran hibaelhárítási célból adatok elemzése szükséges. Nagy mértékben a kézi beavatkozás következtében gyakran szükséges értelmezheti az adatokat, az problémák okait, és javítsa ki azokat a megfelelő stratégiát javasoljuk. Akkor célszerű csupán be ezeket az információkat egy példányát tárolja az eredeti formátumában, és lehetővé teszi egy szakértő cold elemzés céljából.

## <a name="visualizing-data-and-raising-alerts"></a>Adatok megjelenítése és a riasztás kiadását okozó
A felügyeleti rendszer fontos eleme azt a képességet jelent-e az adatok úgy, hogy egy olyan operátort gyorsan ismerhető problémák és a trendeket. Fontos is gyorsan értesítsen operátor, ha egy jelentős esemény történt, amely képes lehet szükség a figyelmet.

Adatok megjelenítési is igénybe vehet néhány űrlapokat, beleértve a képi megjelenítés irányítópultok, használatával, és jelentéskészítési lehetőségei.

### <a name="visualization-by-using-dashboards"></a>Irányítópultok a képi megjelenítés
A leggyakoribb adato módja, így diagramokat, diagramokat vagy valamilyen más ábra sorozataként irányítópulton. Ezek a tételek is paraméteres, és valamelyik elemzőnek válassza ki a fontos paraméterek (például az adott időszakban) bármely adott helyzetnek képesnek kell lennie.

Irányítópultok is hierarchikusan. Legfelső szintű irányítópultok egy átfogó betekintést nyújt a rendszer minden szempontját, de az üzemeltető számára a részletek lebontva. Például egy irányítópultot, amely a rendszer a teljes lemez i/o ábrázol lehetővé kell tennie, hogy valamelyik elemzőnek annak megállapítása, hogy egy minden egyes lemez i/o sebességeit megtekintéséhez, vagy pontosabb eszközök fiókja le aránytalanul nagy mennyiségű forgalom. Ideális esetben az irányítópulton is megjelenjen-e kapcsolódó információkat, például az összes kérelem (a felhasználó vagy a tevékenység), hogy az i/o forrása. Ez az információ felhasználható annak meghatározásához, hogy (és hogyan) való egyenletesen a terhelés több eszközön, és hogy a rendszer hajtaná végre jobban Ha több eszköz sem lett felvéve.

Egy irányítópultot is használható a színkódolás vagy valamilyen más vizuális jelek jelzésére értékek rendellenes megjelenő vagy a várt tartományon kívül. Az előző példa:

* A lemez i/o sebesség mellett, amely hamarosan eléri a maximális kapacitásához hosszabb ideig (gyakran használt adatok lemez) is kiemelten piros.
* A lemez i/o sebesség mellett, amely a rendszeresen rövid időszakokra a maximális korlátját (meleg lemez) sárga színnel is lehet.
* Olyan lemezzel, amelyet a normál használati tapasztal zöld meg lehet jeleníteni.

Vegye figyelembe, hogy egy irányítópult rendszer hatékonyan működjön, a nyers adatok kell rendelkeznie. Ha a saját irányítópult rendszer fejleszt, vagy egy másik szervezet által fejlesztett az irányítópulton, ismernie kell a mely WMI-adatok gyűjtésére, milyen szintű részletességű, és hogyan azt formátumban kell lennie az irányítópult felhasználását.

Egy jó irányítópult nem csak információk jelennek meg, emellett lehetővé teszi egy elemző jelentenek ezt az információt ad hoc kérdésekre. Egyes rendszerek adja meg a felügyeleti eszközök, amelyek operátor ezeket a műveleteket, és áttekintheti az alapul szolgáló adatokat. Azt is megteheti attól függően, hogy a tárházban, amellyel az adatokat, esetleg adatait közvetlenül, vagy importálnia kell azt eszközök például a Microsoft Excel további elemzéshez és jelentéskészítéshez.

> [!NOTE]
> Az arra jogosult személyek irányítópultok hozzáférés korlátozására, mivel előfordulhat, hogy üzleti szempontból érzékeny ezt az információt. Irányítópultok, hogy a felhasználók nem módosíthatják az adatok védelme is kell.
> 
> 

### <a name="raising-alerts"></a>Riasztás kiváltása
Riasztások a figyelés és a rendszerállapot-adatok elemzése és igénylő jelentős esemény történt észlelésekor értesítés létrehozása során a rendszer.

Riasztási biztosíthatja, hogy a rendszer továbbra is működik megfelelően, rugalmas és biztonságos. A rendszer, amely a teljesítményt, rendelkezésre állási és adatvédelmi garanciák teszi a felhasználók számára, ahol módosítania kell az adatokat a azonnal intézkedni fontos része legyen. Operátor módosítania kell a riasztást kiváltó esemény értesíti. Riasztási is használható meghívni a rendszer funkciók, például az automatikus skálázást.

Riasztási általában attól függ, hogy a következő WMI-adatok:

* Biztonsági események. Az eseménynaplók jelzik, amely ismételt hitelesítés és/vagy hitelesítési hibák történnek, a rendszer lehet támadás alatt áll, és operátor tájékoztatni kell.
* Teljesítménymértékeket. A rendszer gyorsan kell válaszolnia, ha egy adott teljesítmény metrika meghaladja a megadott küszöbértéket.
* Rendelkezésre állási adatait. Ha hibát észlel, gyorsan egy vagy több alrendszereket, vagy egy biztonsági mentési erőforrás feladatátvételt szükség lehet. Ismétlődő hibák alrendszernek komoly aggályokat utalhat.

Operátorok riasztási információkat fordulhat elő, például az e-mailek, személyhívó eszköz, vagy SMS szöveg számos kézbesítési csatornák használatával. Előfordulhat, hogy egy riasztás is utalhat, hogy hogyan kritikus olyan helyzet van. Sok riasztási rendszer támogatja az előfizető csoportok, és ugyanabban a csoportban lévő összes operátorok kaphat olyan riasztásokat.

Egy riasztási rendszer testreszabható legyen, és az alapul szolgáló WMI-adatok a megfelelő értéket paraméterként megadható. Ez a megközelítés lehetővé teszi, hogy az adatok szűrése, és arra utalnak, ezeket a küszöbértékeket vagy érdeklődésére számot tartó értékek kombinációi operátor. Vegye figyelembe, hogy bizonyos esetekben a nyers WMI-adatok adható meg a riasztási rendszer. Más helyzetekben akkor célszerű több összesített adatokat. (Például is kell figyelmeztet, ha egy csomópont a CPU-felhasználást 90 százalékára túllépte az elmúlt 10 perc). A riasztási rendszer megadott adatok is tartalmaznia kell a megfelelő összegzése és a helyi adatokat. Ezek az adatok segítségével az esélye, hogy vakriasztások események trip-e a riasztást.

### <a name="reporting"></a>Jelentéskészítés
Jelentéskészítési használja a rendszer általános áttekintést készítéséhez. Akkor lehet, hogy tartalmaznia előzményadatokat aktuális információk mellett. Jelentéskészítési követelmények maguk két nagyobb kategóriába tartoznak: működési jelentése és a biztonsági jelentés.

Működési jelentése többek között a következő szempontokat:

* Statisztikai adatokat tartalmaz, használatával megérthetik, hogy a megadott időszak alatt a rendszer általános vagy megadott alrendszereket erőforrás-használat összesítése
* Az erőforrás-felhasználás a rendszer általános vagy megadott alrendszereket trendek azonosításához egy megadott időszakban
* A kivétel történt a rendszerben vagy a megadott alrendszereket meghatározott időszakon belül figyelése
* Az alkalmazás szempontjából a telepített erőforrások hatékonyságának meghatározása, és az ismertetése, hogy erőforrások mennyisége (és azok kapcsolódó költségét) anélkül, hogy befolyásolná a teljesítmény feleslegesen csökkenthető

Biztonsági reporting aggasztanak nyomon követése az ügyfelek használatára. Ez az alábbiakból állhat:

* A naplózás felhasználói műveletek. Ehhez az egyes kérelmeket a minden felhasználó hajtja végre, és a dátum és idő rögzítése. A adatszerkezet ahhoz, hogy a rendszergazda gyorsan hozza létre újból a felhasználó végző adott időszakon belül műveletek sorrendjét.
* Felhasználó követési erőforrás használja. Ehhez a rögzítés hogyan kér egy felhasználó hozzáfér a különböző erőforrások, a rendszer alkotó, valamint arról, hogyan hosszú. A rendszergazda felhasználó kihasználtsági jelentés készítése egy adott időszakban, valószínűleg számlázási okokból használhatja ezeket az adatokat kell lennie.

Sok esetben kötegelt folyamatok jelentéskészítés meghatározott ütemezés szerint. (Késés általában nem kapcsolatos problémát.) De ezek is elérhetőnek kell lenniük generációs eseti alapon szükség esetén. Tegyük fel Ha egy relációs adatbázisban, például az Azure SQL Database adatokat tároló segítségével olyan eszköz, például az SQL Server Reporting Services kinyerése és adatok formázása és jelenthet, mint a jelentések.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták
* [Automatikus skálázás útmutatást](../best-practices/auto-scaling.md) ismerteti, hogyan lehet felügyeleti terhelés csökkentése így a folyamatosan figyelje a rendszer teljesítményét és megalapozott döntéseket hozzáadásával vagy eltávolításával erőforrások az operátor.
* [Állapotfigyelő végpont figyelési mintát](https://msdn.microsoft.com/library/dn589789.aspx) ismerteti, hogyan lehet olyan alkalmazás, amely a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető funkcionális ellenőrzés végrehajtásához.
* [Prioritás várólista mintát](https://msdn.microsoft.com/library/dn589794.aspx) bemutatja, hogyan várólistára helyezett üzenetek rangsorolására, hogy a sürgős kérelmek fogadásának és feldolgozhatók üzenetfejléceket üzenetek előtt.

## <a name="more-information"></a>További információ
* [Figyelése, diagnosztizálása és elhárítása a Microsoft Azure tárolás](/azure/storage/storage-monitoring-diagnosing-troubleshooting)
* [Azure: Telemetriai alapjai és hibaelhárítás](http://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx)
* [Azure Cloud Services és a virtuális gépek diagnosztika engedélyezése](/azure/cloud-services/cloud-services-dotnet-diagnostics)
* [Azure Redis gyorsítótár](https://azure.microsoft.com/services/cache/), [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/), és [HDInsight](https://azure.microsoft.com/services/hdinsight/)
* [A Service Bus-üzenetsorok használata](/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)
* [SQL Server üzleti intelligencia Azure virtuális gépek](/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-bi)
* [Riasztási értesítéseket](/azure/monitoring-and-diagnostics/insights-receive-alert-notifications) és [nyomon követése szolgáltatásának állapota](/azure/monitoring-and-diagnostics/insights-service-health)
* [Application Insights](/azure/application-insights/app-insights-overview)

