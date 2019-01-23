---
title: Monitorozási és diagnosztikai útmutató
titleSuffix: Best practices for cloud applications
description: Ajánlott eljárások a felhőben futó elosztott alkalmazások monitorozásához.
author: dragon119
ms.date: 07/13/2016
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: fa4ed5fde2e52e87763c1b528661aea1b7f90bcf
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485009"
---
# <a name="monitoring-and-diagnostics"></a>Monitorozás és diagnosztika

A felhőben futó elosztott alkalmazások és szolgáltatások jellegükből adódóan összetett szoftverek, amelyek számos mozgó részből tevődnek össze. Éles környezetben fontos a rendszer felhasználók általi használatának nyomon követhetősége, az erőforrás-felhasználás nyomon követhetősége, valamint a rendszer állapotának és teljesítményének általános monitorozhatósága. Ezeknek a diagnosztikai információknak a segítségével észlelheti és javíthatja a problémákat, valamint felderítheti a potenciális problémákat, és megelőzheti azok bekövetkezését.

## <a name="monitoring-and-diagnostics-scenarios"></a>Monitorozási és diagnosztikai forgatókönyvek

A monitorozás segítségével betekintést nyerhet abba, hogy mennyire jól működik a rendszer. A monitorozás kritikus szerepet játszik a szolgáltatásminőségi célok elérésében. Általános forgatókönyvek a monitorozási adatok gyűjtésére:

- A rendszer megfelelő állapotának biztosítása.
- A rendszer és összetevői rendelkezésre állásának nyomon követése.
- A megfelelő teljesítmény fenntartása, hogy a rendszer feldolgozási képessége ne romoljon le váratlanul a munkamennyiség növekedésével.
- Annak szavatolása, hogy a rendszer megfeleljen az ügyfelekkel kötött szolgáltatásiszint-szerződéseknek (service-level agreement, SLA).
- A rendszer, a felhasználók és a felhasználói adatok védelmének és biztonságának biztosítása.
- A naplózási és szabályozási céllal végzett műveletek nyomon követése.
- A rendszer mindennapos használatának nyomon követése, és a trendek észlelése, amelyek megfelelő ellenintézkedések hiányában problémákat okozhatnak.
- A felmerülő hibák nyomon követése a kezdeti jelentéstől a lehetséges okok elemzéséig, javításig, a hibákból fakadó szoftverfrissítésekig és üzembe helyezésig.
- A műveletek nyomon követése, és a szoftverkiadások hibáinak elhárítása.

> [!NOTE]
> A fenti lista nem tekintendő átfogónak. A dokumentum ezekre a forgatókönyvekre, mint a leggyakoribb monitorozási helyzetekre összpontosít. Természetesen lehetnek más forgatókönyvek is, amelyek kevésé gyakoriak, vagy csak az adott környezetre jellemzőek.

Az alábbi szakaszokban részletesebben tárgyaljuk ezeket a forgatókönyveket. Az egyes forgatókönyvekkel kapcsolatos információkat az alábbi formában ismertetjük:

1. A forgatókönyv rövid áttekintése.
2. A forgatókönyv jellemző követelményei.
3. A forgatókönyv végrehajtásához szükséges nyers rendszerállapot-adatok, valamint ezek lehetséges forrásai.
4. A nyers adatok elemzésének és egyesítésének módszerei hasznos diagnosztikai információk létrehozása céljából.

## <a name="health-monitoring"></a>Állapotfigyelés

A rendszer állapota akkor megfelelő, ha a rendszer fut, és képes kéréseket feldolgozni. Az állapotmonitorozás célja, hogy egy pillanatképet készítsen a rendszer aktuális állapotáról, amely alapján ellenőrizheti, hogy a rendszer minden összetevője a várt módon működik-e.

### <a name="requirements-for-health-monitoring"></a>Az állapotmonitorozásra vonatkozó követelmények

Az operátoroknak gyorsan (pár másodpercen belül) értesülnie kell, ha a rendszer bármely részének működése nem megfelelőnek tekinthető. Az operátornak meg kell tudnia állapítani, hogy a rendszer mely részei működnek megfelelően, és mely részekben tapasztalhatók problémák. A rendszerállapot jelzőlámpás megoldással jeleníthető meg:

- Vörös jelzi a meghibásodást (a rendszer leállt).
- Sárga jelzi a részleges meghibásodást (a rendszer kevesebb funkciót biztosítva fut).
- Zöld jelzi a teljesen kifogástalan működést.

Az átfogó állapotmonitorozási rendszer segítségével az operátor részletesen elemezheti a rendszer állapotát egészen az alrendszerek és az összetevők állapotáig. Ha például a rendszer általános állapota részlegesen megfelelőként jelenik meg, az operátornak képesnek kell lennie a nagyításra, és annak megállapítására, hogy épp melyik funkció nem érhető el.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

Az állapotmonitorozás támogatásához szükséges nyers adatok a következők eredményeként hozhatók létre:

- A felhasználói kérések végrehajtásának nyomon követése. Ezeknek az információknak a segítségével határozható meg, hogy mely kérések voltak sikeresek és melyek nem, és hogy mennyi ideig tart az egyes kérések végrehajtása.
- Szintetikus felhasználómonitorozás. Ez a folyamat a felhasználó által végrehajtott lépéseket szimulálja, és előre meghatározott lépések sorát követi. Az egyes lépések eredményét rögzíteni kell.
- Kivételek, hibák és figyelmeztetések naplózása. Ezek az információk az alkalmazás kódjába ágyazott nyomkövetési utasítások eredményeként, valamint a rendszer által hivatkozott szolgáltatások eseménynaplóiból lekért adatok alapján rögzíthetők.
- A rendszer által használt külső szolgáltatások állapotának monitorozása. Az ilyen monitorozás során szükség lehet az érintett szolgáltatások által biztosított állapotadatok lekérésére és elemzésére. Ezek az adatok számos különféle formátumban fordulhatnak elő.
- Végpont-monitorozás. Ezt a mechanizmust „A rendelkezésre állás monitorozása” című szakasz ismerteti részletesebben.
- Környezeti teljesítményadatok gyűjtése, például a háttérbeli processzorhasználatra vagy I/O-tevékenységekre (beleértve a hálózati tevékenységeket is) vonatkozóan.

### <a name="analyzing-health-data"></a>Állapotadatok elemzése

Az állapotmonitorozás elsődleges célja annak gyors jelzése, hogy a rendszer fut-e. A közvetlen adatok azonnali elemzése riasztást válthat ki, ha egy kritikus összetevő állapotát a monitorozás nem megfelelőnek ítéli. (Például akkor, amikor nem válaszol több egymást követő pingelésre.) Az operátor így végrehajthatja a megfelelő korrekciós műveleteket.

A fejlettebb rendszerek előrejelzési elemmel is rendelkezhetnek, amelyek a legutóbbi és az aktuális számítási feladatok hideg elemzése alapján működnek. A hideg elemzés képes észlelni a tendenciákat, és meghatározni, hogy a rendszer várhatóan megfelelő állapotú marad-e, és hogy van-e szüksége további erőforrásokra. Az előrejelzési elemnek kritikus teljesítménymutatók alapján kell működnie, például:

- Az egyes szolgáltatásokra vagy alrendszerekre irányuló kérések mennyisége.
- Az ilyen kérések válaszideje.
- Az egyes szolgáltatásokba be- és onnan kilépő adatok mennyisége.

Ha valamelyik metrika értéke meghaladja a meghatározott küszöbértéket, a rendszer riasztást küldhet, hogy az operátor vagy az automatikus skálázás (ha van) megtehesse a rendszer megfelelő állapotának fenntartásához szükséges megelőző intézkedéseket. Ilyen intézkedések lehetnek az erőforrások hozzáadása, egy vagy több meghibásodott szolgáltatás újraindítása, vagy az alacsonyabb prioritású kérések szabályozása.

## <a name="availability-monitoring"></a>A rendelkezésre állás monitorozása

Egy valóban kifogástalan állapotú rendszerhez elengedhetetlen, hogy a rendszert alkotó összetevők és alrendszerek rendelkezésre álljanak. A rendelkezésre állás monitorozása szorosan kapcsolódik az állapotmonitorozáshoz. Amíg azonban az állapotmonitorozás a rendszer aktuális állapotának azonnali helyzetét jeleníti meg, a rendelkezésre állás monitorozása a rendszer és a rendszer összetevőinek rendelkezésre állását követi nyomon, és statisztikákat készít a rendszer üzemidejére vonatkozóan.

Számos rendszerben bizonyos összetevők (például az adatbázisok) beépített redundanciával vannak konfigurálva, így gyors feladatátvételt tesznek lehetővé súlyos hibák vagy a kapcsolat megszakadása esetén. Ideális esetben a felhasználók nem is tudhatják, hogy ilyen hiba történt. A rendelkezésre állás monitorozása szempontjából azonban a lehető legtöbb információt kell gyűjteni az ilyen hibákkal kapcsolatban, hogy megállapíthatók legyenek az okok, és végre lehessen hajtani a korrekciós műveleteket a hibák ismételt előfordulásának elkerülésére.

A rendelkezésre állás nyomon követéséhez szükséges adatok számos alacsonyabb szintű tényezőtől függhetnek. Ezeknek a tényezőknek jelentős része az adott alkalmazásra, rendszerre vagy környezetre lehet jellemző. A hatékony monitorozási rendszerek az ezeknek az alacsony szintű tényezőknek megfelelő rendelkezésreállási adatokat rögzítik, majd azokat összesítve átfogó képet biztosítanak a rendszerről. Egy e-kereskedelmi rendszerben például az ügyfelek általi rendelést lehetővé tevő üzleti funkció függhet a megrendelések adatait tároló adattártól, valamint a megrendelések kifizetésére vonatkozó pénzügyi tranzakciókat kezelő fizetési rendszertől. A rendszer rendelési részének rendelkezésre állása ezért az adattár és a fizetési alrendszer rendelkezésre állásának függvénye.

### <a name="requirements-for-availability-monitoring"></a>A rendelkezésre állás monitorozására vonatkozó követelmények

Az operátornak meg kell tudnia tekinteni az egyes rendszerek és alrendszerek rendelkezésreállási előzményadatait is, és ezeknek az információknak a segítségével ki kell tudniuk mutatni a tendenciákat, amelyek egy vagy több alrendszer rendszeres meghibásodását okozhatják. (A szolgáltatások egy olyan adott napszakban kezdenek meghibásodni, amely a feldolgozási csúcsidőszaknak felel meg?)

A monitorozási megoldásoknak azonnali és előzményadatokat tartalmazó nézeteket is biztosítaniuk kell az egyes alrendszerek rendelkezésre állásával vagy rendelkezésre nem állásával kapcsolatban. Emellett képesnek kell lennie gyorsan riasztani az operátort, amikor egy vagy több szolgáltatás meghibásodik, vagy ha a felhasználók nem érik el a szolgáltatásokat. Ehhez nem csupán az egyes szolgáltatások monitorozása szükséges, hanem a felhasználók által végrehajtott műveletek vizsgálata is, ha ezek a műveletek meghiúsulnak, amikor kommunikálni próbálnak a szolgáltatással. A csatlakozási hibák bizonyos mértékig normálisnak tekinthetők, és átmeneti hibák miatt léphetnek fel. Hasznos lehet azonban engedélyezni a rendszer számára, hogy riasztást küldjön egy adott alrendszerhez kapcsolódóan egy adott időszakban fellépő kapcsolathibák számáról.

<!-- markdownlint-disable MD024 MD033 -->

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

Az állapotmonitorozáshoz hasonlóan, a rendelkezésre állás monitorozásának a támogatásához szükséges nyers adatok is a szintetikus felhasználómonitorozás, valamint a kivételek, hibák és figyelmeztetések naplózásának az eredményeként hozhatók létre. A rendelkezésreállási adatok emellett végpont-monitorozásból is beszerezhetők. Az alkalmazás egy vagy több állapotvégpontot is elérhetővé tehet a rendszer egy funkcionális területe elérésének teszteléséhez. A monitorozási rendszer pingelheti az egyes végpontokat egy meghatározott ütemezés szerint, és gyűjtheti ennek eredményét (sikeres vagy sikertelen).

Az összes időtúllépést, hálózati kapcsolathibát és újrapróbált kapcsolódást rögzíteni kell. Minden adatot időbélyeggel kell ellátni.

### <a name="analyzing-availability-data"></a>A rendelkezésreállási adatok elemzése

A rendszerállapot-adatokat összesíteni és korrelálni kell a következő elemzéstípusok támogatásához:

- A rendszer és az alrendszerek közvetlen rendelkezésre állása.
- A rendszer és az alrendszerek rendelkezésreállási hibáinak aránya. Ideális esetben az operátornak képesnek kell lennie a hibák korrelálására az adott tevékenységekkel: mi történt, amikor a rendszer meghibásodott?
- A rendszer vagy az alrendszerek meghibásodási arányainak előzménynézete adott időszakra vonatkozóan, valamint a rendszer terhelése (például a felhasználói kérések száma) a hiba bekövetkezésekor.
- A rendszer vagy egy alrendszer rendelkezésre nem állásának okai. Ilyen ok lehet például, ha a szolgáltatás nem fut, ha megszakad a kapcsolat, ha létrejön a kapcsolat, de időtúllépés történik, vagy ha létrejön a kapcsolat, de hibát ad vissza.

A szolgáltatások adott időszakra vetített százalékos rendelkezésre állását az alábbi képlettel számíthatja ki:

```console
%Availability =  ((Total Time – Total Downtime) / Total Time ) * 100
```

Ez az SLA szempontjából hasznos. (Az [SLA monitorozását](#sla-monitoring) az útmutató későbbi része ismerteti részletesen.) Az *állásidő* definíciója a szolgáltatástól függ. A Visual Studio Team Services Build Service például azon időtartamként (halmozott percek teljes száma) határozza meg az állásidőt, ameddig a Build Service nem állt rendelkezésre. Egy perc akkor tekintendő rendelkezésre nem állónak, ha az adott percben a Build Service számára küldött, ügyfél által kezdeményezett műveletek végrehajtására irányuló, egymást követő HTTP-kérések hibakódot eredményeznek vagy nem adnak vissza választ.

## <a name="performance-monitoring"></a>Teljesítményfigyelés

A rendszer egyre nagyobb terhelése esetén (a felhasználók mennyiségének növekedésével), a felhasználók által elért adathalmazok mérete egyre nő, és egyre valószínűbbé válik egy vagy több összetevő meghibásodása. Az összetevők meghibásodását gyakran a teljesítmény csökkenése előzi meg. Ha képes észlelni az ilyen teljesítménycsökkenéseket, proaktív lépéseket tehet a helyzet orvoslására.

A rendszer teljesítménye számos tényezőtől függ. Az egyes tényezőket általában fő teljesítménymutatókkal (KPI) mérik, ilyen például az adatbázis-tranzakciók másodpercenkénti száma vagy a sikeresen kiszolgált hálózati kérések mennyisége egy adott időszakban. A KPI-k némelyike önálló teljesítménymutatóként érhető el, mások mérőszámok kombinációjából állnak.

> [!NOTE]
> A gyenge vagy a jó teljesítmény meghatározásához ismerni kell azt a teljesítményszintet, amelyen a rendszernek képesnek kell lennie üzemelni. Ehhez meg kell figyelni a rendszert, miközben átlagos terhelés mellett működik, és rögzíteni kell az egyes KPI-k adatait egy időszakra vonatkozóan. Ez a rendszernek szimulált terhelés mellett, tesztkörnyezetben történő futtatásával, és a vonatkozó adatok gyűjtésével is járhat, mielőtt éles környezetben telepítené a rendszert.
>
> Gondoskodnia kell arról, hogy a teljesítmény monitorozása ne váljon teherré a rendszer számára. A teljesítménymonitorozási folyamat által gyűjtött adatok részletességi szintjét érdemes lehet dinamikusan változtathatóvá tenni.

### <a name="requirements-for-performance-monitoring"></a>A teljesítmény monitorozására vonatkozó követelmények

A rendszer teljesítményének vizsgálatához az operátornak általában az alábbi információkra van szüksége:

- A felhasználói kérések megválaszolásának sebessége.
- Az egyidejű felhasználói kérések száma.
- A hálózati forgalom mennyisége.
- Az üzleti tranzakciók teljesítésének sebessége.
- A kérések átlagos feldolgozási ideje.

Hasznos lehet olyan eszközökkel is ellátni az operátorokat, amelyek segítenek kimutatni az alábbiak közötti összefüggéseket, például:

- Az egyidejű felhasználók száma és a kérések késése (mennyi ideig tart egy kérés feldolgozásának a megkezdése, miután a felhasználó elküldte azt).
- Az egyidejű felhasználók száma és az átlagos válaszidő (mennyi ideig tart egy kérés befejezése, miután elkezdődött annak feldolgozása).
- A kérések mennyisége és a feldolgozási hibák száma.

Az ilyen magas szintű működési információk mellett az operátornak a rendszer egyes összetevőinek a teljesítményéről is részletes képet kell kapnia. Ezek az adatok általában alsó szintű teljesítményszámlálók használatával biztosíthatók, amelyek az alábbi információkat követik nyomon:

- Memóriahasználat.
- Szálak száma.
- Processzor feldolgozási ideje.
- Kérésvárólista hossza.
- Lemez vagy hálózat I/O-sebessége és -hibái.
- Az írt vagy olvasott bájtok száma.
- Köztes szoftverek mutatói, például az üzenetsor hossza.

Minden megjelenítésnek lehetővé kell tennie az operátor számára egy időtartam megadását. A megjelenített adatok az aktuális helyzet pillanatképe és/vagy a teljesítmény előzménynézete lehetnek.

Az operátornak tudnia kell riasztást küldenie bármely megadott érték tetszőleges időintervallumra vonatkozó teljesítménymutatója alapján.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A rendszerre érkező és azon áthaladó felhasználói kérések állapotának monitorozásával magas szintű teljesítményadatokat gyűjthet (átviteli sebesség, egyidejű felhasználók száma, üzleti tranzakciók száma, hibaarányok és hasonlók). Ehhez nyomkövetési utasításokat kell az alkalmazáskód kulcspontjain elhelyezni időzítési információkkal együtt. Minden hibát, kivételt és figyelmeztetést elegendő mennyiségű adattal kell rögzíteni az őket okozó kérésekkel való korrelálás érdekében. Az Internet Information Services (IIS) naplója egy másik hasznos forrás.

Amennyiben lehetséges, az alkalmazás által használt külső rendszerek teljesítményadatait is rögzíteni kell. A külső rendszerek biztosíthatnak saját teljesítményszámlálókat vagy egyéb funkciókat a teljesítményadatok lekéréséhez. Ha ez nem lehetséges, rögzítse az olyan információkat, mint a külső rendszerre felé irányuló kérések kezdési és befejezési időpontja, valamint a művelet állapota (sikeres, sikertelen vagy figyelmeztetés). A kérésidők méréséhez használhat például egy stopperóra jellegű módszert: a kérés indításakor indítson el egy időmérőt, a befejezésekor pedig állítsa le.

A rendszerek egyes összetevőinek alacsony szintű teljesítményadatai olyan funkciókon és szolgáltatásokon keresztül érhetők el, mint a Windows-teljesítményszámlálók és az Azure Diagnostics.

### <a name="analyzing-performance-data"></a>Teljesítményadatok elemzése

Az elemzési munka nagy részét a teljesítményadatoknak a felhasználói kérés típusa és/vagy azon alrendszer vagy szolgáltatás alapján való összesítése teszi ki, amely számára az egyes kérések el lettek küldve. Felhasználói kérés lehet például egy cikk hozzáadása egy bevásárlókosárhoz vagy a fizetési folyamat végrehajtása egy e-kereskedelmi rendszerben.

Egy másik gyakori követelmény a kiválasztott percentilisekben található teljesítményadatok összegzése. Az operátor például meghatározhatja a kérések 99 százalékának, a kérések 95 százalékának és a kérések 70 százalékának a válaszidejét. Az egyes percentilisekre SLA-célok vagy más célkitűzések vonatkozhatnak. A folyamatos eredményeket közel valós időben kell jelenteni, hogy észlelhetők legyenek az azonnali problémák. Az eredményeket emellett hosszabb időre vonatkozóan is összesíteni kell statisztikai célokból.

Amennyiben késési problémák vetnék vissza a teljesítményt, az operátornak gyorsan kell tudnia azonosítani a szűk keresztmetszet okát az egyes kérések által végrehajtott egyes lépések késésének vizsgálatával. A teljesítményadatoknak így lehetőséget kell biztosítaniuk az egyes lépések teljesítménymutatóinak korrelálására, hogy az adott kérésekhez lehessen kötni azokat.

A megjelenítés követelményeitől függően érdemes lehet létrehozni és menteni egy adatkockát, amely a nyers adatok nézeteit tartalmazza. Az adatkocka lehetővé teszi a teljesítményadatok komplex ad hoc lekérdezését és elemzését.

## <a name="security-monitoring"></a>A biztonság monitorozása

A bizalmas adatokat kezelő kereskedelmi rendszerek mindegyikében meg kell valósítani egy biztonsági struktúrát. A biztonsági mechanizmus összetettségét általában az adatok bizalmassága határozza meg. A felhasználói hitelesítést igénylő rendszerekben az alábbiakat kell rögzíteni:

- Az összes bejelentkezési kísérletet, függetlenül attól, hogy sikeresek vagy sikertelenek voltak.
- Által végrehajtott összes műveletet &mdash; és által elért összes erőforrás részleteit &mdash; egy hitelesített felhasználó.
- A munkamenet felhasználó általi befejezésének és a felhasználó kijelentkezésének időpontja.

A monitorozás segítséget nyújthat a rendszert érő támadások észlelésében. Ha például magas a sikertelen bejelentkezési kísérletek száma, az találgatásos támadásra utalhat. A kérések számának váratlan megugrása elosztott szolgáltatásmegtagadásos (DDoS-) támadás eredménye lehet. Fel kell készülnie arra, hogy az erőforrások mindegyikére irányuló összes kérést monitorozza azok forrásától függetlenül. A bejelentkezéshez kapcsolódó biztonsági réssel rendelkező rendszerek véletlenül elérhetővé tehetik az erőforrásokat a külvilág számára anélkül, hogy a felhasználónak ténylegesen be kellene jelentkeznie.

### <a name="requirements-for-security-monitoring"></a>A biztonság monitorozására vonatkozó követelmények

A biztonság monitorozásának legfontosabb szempontjait figyelembe véve, az operátornak képesnek kell lennie gyorsan:

- észlelni a nem hitelesített entitások általi behatolási kísérleteket;
- azonosítani az entitások kísérleteit olyan adatokra irányuló műveletek végrehajtására, amelyekhez az entitások nem rendelkeznek hozzáféréssel;
- megállapítani, ha a rendszer vagy annak egy része külső vagy belső támadás alatt áll. (Például egy rosszindulatú hitelesített felhasználó megpróbálja leállítani a rendszert.)

Ezeknek a követelményeknek a kiszolgálása érdekében az operátort értesíteni kell:

- Ha egy fiók sorozatos sikertelen bejelentkezési kísérleteket tesz egy meghatározott időszakon belül.
- Ha egy hitelesített fiók sorozatosan tiltott erőforrásokhoz próbál hozzáférni egy meghatározott időszakon belül.
- Ha nagy számú nem hitelesített vagy jogosulatlan kérés érkezik egy meghatározott időszakon belül.

Az operátor rendelkezésére bocsátott információknak tartalmaznia kell az egyes kérések forrásának gazdagépcímét. Ha egy adott címtartománnyal kapcsolatban rendszeresen merülnek fel biztonsági megsértések, ezeket a gazdagépeket érdemes lehet blokkolni.

A rendszer biztonságának fenntartásában kulcsszerepet játszik a szokásos mintáktól eltérő műveletek gyors észlelése. A sikertelen és/vagy sikeres bejelentkezési kérések számára vonatkozó információk vizuális megjelenítésével például könnyebben észlelhető, ha szokatlan időpontban hajtanak végre nagy számú tevékenységet. (Például a felhasználók hajnali 3:00 órakor bejelentkeznek, és nagy számú műveletet hajtanak végre, holott a munkaidejük csak reggel 9:00 órakor kezdődik.) Ezen információkkal egyszerűbben konfigurálható az időalapú automatikus skálázás is. Ha például az operátor észreveszi, hogy sok felhasználó jelentkezik be rendszeresen egy adott napszakban, további hitelesítési szolgáltatásokat indíthat el a megnövekedett munkamennyiség kezelésére, majd leállíthatja a szolgáltatások a csúcsidőszak végét követően.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A legtöbb elosztott rendszer esetében a biztonság mindenre kiterjedő szempont. A kapcsolódó adatok létrehozása valószínűleg a rendszer több pontján történik. Érdemes lehet bevezetni egy biztonságiadat- és eseménykezelési (Security Information and Event Management, SIEM) megközelítést az alkalmazás, a hálózati berendezések, a kiszolgálók, a tűzfalak, a víruskereső szoftverek és az egyéb behatolásmegelőzési elemek által létrehozott eseményekből eredő, biztonsággal kapcsolatos adatok gyűjtésére.

A biztonság monitorozása az alkalmazás részét nem képező eszközökről származó adatokra is kiterjedhet. Ezek az eszközök lehetnek a külső ügynökségek portkeresési tevékenységeit azonosító segédprogramok, vagy az alkalmazásra és az adatokra irányuló, nem hitelesített hozzáférésre tett kísérleteket észlelő hálózati szűrők.

Bármiről is legyen szó, az összegyűjtött adatok segítségével a rendszergazda meghatározhatja a támadás természetét, és megteheti a megfelelő ellenintézkedéseket.

### <a name="analyzing-security-data"></a>Biztonsággal kapcsolatos adatok elemzése

A biztonság monitorozásának egyik jellemzője, hogy az adatok rengeteg különböző forrásból származhatnak. A különböző formátumok és részletességi szintek miatt gyakran szükség van a rögzített adatok összetett elemzésére és egy összefüggő információszállá való összefűzésükre. A legegyszerűbb esetektől (például a nagyszámú sikertelen bejelentkezéstől, vagy a kritikus erőforrásokhoz való sorozatos, jogosulatlan hozzáférésre tett kísérletek észlelésétől) eltekintve előfordulhat, hogy a biztonsággal kapcsolatos adatok bonyolult automatikus feldolgozása nem lehetséges. Ezeket az adatokat inkább időbélyeggel ellátva, de egyébként eredeti formájukban egy biztonságos adattárba érdemes írni a manuális szakértői elemzés lehetővé tétele érdekében.

## <a name="sla-monitoring"></a>Az SLA monitorozása

A fizető ügyfeleket kiszolgáló számos kereskedelmi rendszer SLA-k formájában szavatolja a rendszer teljesítményét. Az SLA-k lényegében azt jelentik ki, hogy a rendszer képes egy meghatározott munkamennyiséget kezelni egy egyeztetett időkereten belül, a kritikus adatok elvesztése nélkül. Az SLA monitorozása annak biztosítására irányul, hogy a rendszer megfelel a mérhető SLA-knak.

> [!NOTE]
> Az SLA monitorozása szorosan kapcsolódik a teljesítmény monitorozásához. Amíg azonban a teljesítmény monitorozásának célja a rendszer *optimális* működésének biztosítása, az SLA monitorozását egy szerződéses kötelezettség szabályozza, amely meghatározza, hogy az *optimális* ténylegesen mit is jelent.

Az SLA-kat általában az alábbiak vonatkozásában határozzák meg:

- A rendszer általános rendelkezésre állása. Előfordulhat például, hogy egy szolgáltató a rendszer 99,9%-os rendelkezésre állását szavatolja. Ez azt jelenti, hogy a rendszer évente legfeljebb 9 órát állhat, azaz hetente körülbelül 10 percet.
- Működési teljesítmény. Ezt általában egy vagy több felső határértékkel fejezik ki, például annak szavatolásával, hogy a rendszer képes akár 100 000 egyidejű felhasználói kérés támogatására, vagy 10 000 egyidejű üzleti tranzakció kezelésére.
- Működési válaszidő. A rendszer a kérések feldolgozásának sebességére is vállalhat kötelezettséget. Például: az üzleti tranzakciók 99 százalékát a rendszer 2 másodpercen belül feldolgozza, és egyetlen tranzakció feldolgozása sem tart 10 másodpercnél hosszabb ideig.

> [!NOTE]
> Egyes kereskedelmi rendszerek szerződései az ügyfélszolgálatra vonatkozó SLA-kat is tartalmazhatnak. Például: minden ügyfélszolgálati kérésre 5 percen belül érkezik válasz, és az ügyfélszolgálat a hibák 99%-át 1 munkanapon belül teljes mértékben elhárítja. A hatékony [problémakövetés](#issue-tracking) (a jelen szakasz egy későbbi része ismerteti) kritikus fontosságú az ezekhez hasonló SLA-k eléréséhez.

### <a name="requirements-for-sla-monitoring"></a>Az SLA monitorozására vonatkozó követelmények

A legfelső szinten az operátornak egy pillantásra meg kell tudnia állapítani, hogy a rendszer teljesíti-e a megállapodásba foglalt SLA-kat vagy sem. Ha nem, az operátornak képesnek kell lennie részletes elemzést végezni, és megvizsgálni a mögöttes tényezőket a teljesítménycsökkenés okainak meghatározásához.

A vizuálisan ábrázolható, jellemző magas szintű mutatók az alábbiak lehetnek:

- A hasznos üzemidő százalékos aránya.
- Az alkalmazás átviteli sebessége (a másodpercenkénti sikeres tranzakciók és/vagy műveletek mennyiségében kifejezve).
- A sikeres/sikertelen alkalmazáskérések száma.
- Az alkalmazás- és rendszerhibák, kivételek és figyelmeztetések száma.

A mutatók mindegyikének szűrhetőnek kell lennie adott időszak alapján.

A felhőalkalmazások vélhetően több alrendszerből és összetevőből állnak. Az operátornak az egyes felső szintű mutatókat kiválasztva meg kell tudnia tekinteni, hogy a mutató hogyan tevődik össze a mögöttes elemek állapota alapján. Ha például a teljes rendszer üzemideje egy elfogadható érték alá csökken, az operátornak képesnek kell lennie a nagyításra, és annak megállapítására, hogy mely elemek járultak hozzá a hibához.

> [!NOTE]
> A rendszer üzemidejét gondosan kell meghatározni. A maximális rendelkezésre állást redundancia használatával biztosító rendszerekben az elemek egyes példányai meghibásodhatnak ugyan, a rendszer azonban továbbra is működőképes marad. A rendszer az állapotmonitorozás által kimutatott üzemidejének az egyes elemek összesített üzemidejét kell mutatnia, és nem feltétlenül azt, hogy a rendszer ténylegesen leállt-e. Emellett a hibák elkülöníthetők. Így ha egy adott alrendszer nem is áll rendelkezésre, a rendszer többi része továbbra is rendelkezésre állhat, jóllehet kevesebb funkciót biztosítva. (Egy e-kereskedelmi rendszerben például az egyik alrendszer hibája miatt a vevő esetleg nem küldhet megrendelést, azonban a termékkatalógust továbbra is böngészheti.)

Riasztási célokból a rendszernek képesnek kell lennie eseményt létrehozni, ha a felső szintű mutatók bármelyike meghalad egy adott küszöbértéket. A felső szintű mutatókat alkotó különféle összetevők alacsonyabb szintű adatainak környezetfüggő adatként elérhetőnek kell lenniük a riasztást létrehozó rendszer számára.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

Az SLA monitorozásának támogatásához szükséges nyers adatok hasonlóak a teljesítmény, és bizonyos szempontokból az állapot és a rendelkezésre állás monitorozásának nyers adataihoz. (További részletekért tekintse meg a vonatkozó szakaszokat.) Az adatok az alábbiak végrehajtásával rögzíthetők:

- A végpontok monitorozása.
- Kivételek, hibák és figyelmeztetések naplózása.
- A felhasználói kérések végrehajtásának nyomon követése.
- A rendszer által használt külső szolgáltatások rendelkezésre állásának monitorozása.
- Teljesítmény-mérőszámok és -számlálók használata.

Minden adatot időponthoz kell kötni, és időbélyeggel kell ellátni.

### <a name="analyzing-sla-data"></a>Az SLA-adatok elemzése

A rendszerállapot-adatokat összesíteni kell a rendszer átfogó teljesítményének megjelenítéséhez. Az összesített adatoknak a részletes elemzést is lehetővé kell tenniük a mögöttes alrendszerek teljesítményének vizsgálatához. Például az alábbiakra kell képesnek lennie:

- Az adott időszakban kezdeményezett felhasználói kérések teljes számának megállapítása, valamint a sikeres és sikertelen kérések arányának meghatározása.
- Átfogó kép kialakítása a rendszer válaszidejéről a felhasználói kérések válaszidejének összesítésével.
- A felhasználói kérések állapotnak elemzése a kérések átfogó válaszidejének a kérést alkotó egyes munkaelemek válaszidejére való lebontásához.
- A rendszer átfogó rendelkezésre állásának meghatározása az üzemidőt százalékos arányként kifejezve egy adott időszakra vonatkozóan.
- A rendszer egyes összetevőinek és szolgáltatásainak az idő százalékában kifejezett rendelkezésre állásának elemzése. Ehhez esetleg szükség lehet a külső szolgáltatások által létrehozott naplók elemzésére.

Számos kereskedelmi rendszernek jelentenie kell a tényleges teljesítményadatokat a megállapodásban foglalt SLA-k alapján egy adott időtartamra, általában egy hónapra vonatkozóan. Az információk segítségével számítható ki az ügyfeleket illető jóváírás vagy egyéb visszatérítés, ha az SLA-k nem teljesültek az adott időszakban. Az egyes szolgáltatások rendelkezésre állását [A rendelkezésreállási adatok elemzése](#analyzing-availability-data) című szakaszban bemutatott módszerrel számíthatja ki.

A vállalatok a szolgáltatások meghibásodását okozó incidensek számát és természetét is követhetik belső használatra. A problémák gyors megoldásának vagy teljes kiküszöbölésének elsajátításával csökkenthető az állásidő, és teljesíthetők az SLA-k.

## <a name="auditing"></a>Naplózás

Az alkalmazás jellegétől függően a törvényi vagy egyéb jogi rendelkezések megszabhatnak a felhasználói műveletek naplózására és az összes adatelérés rögzítésére vonatkozó követelményeket. A naplózás az ügyfeleket az egyes kérésekhez kötő bizonyítékot biztosíthat. A letagadhatatlanság fontos tényező számos elektronikus üzleti rendszerben az ügyfél és az alkalmazásért vagy szolgáltatásért felelős vállalat közötti bizalom fenntartásában.

### <a name="requirements-for-auditing"></a>A naplózásra vonatkozó követelmények

Az elemzőknek nyomon kell tudniuk követni a felhasználók által végrehajtott üzleti műveletek sorát, hogy rekonstruálhassák a felhasználók műveleteit. Előfordulhat, hogy erre egyszerűen a nyilvántartás miatt van szükség, vagy pedig egy törvényszéki vizsgálat részeként.

A naplóadatok rendkívül bizalmasak. Valószínűleg olyan adatokat is tartalmaznak, amelyek azonosítják a rendszer felhasználóit és az általuk végrehajtott feladatokat. A naplóadatok emiatt leginkább a kizárólag a megbízható elemzők számára elérhető jelentések formájában állnak rendelkezésre, nem pedig a grafikai műveletek részletes elemzését lehetővé tévő interaktív rendszerként. Az elemzőnek különböző jelentéseket kell tudniuk létrehozni. A jelentések például listázhatják az adott időszakban bekövetkezett felhasználó tevékenységeket, részletes kronológiát biztosíthatnak egy adott felhasználó tevékenységéről, vagy listázhatják az egy vagy több erőforrásra vonatkozóan végrehajtott műveletek sorát.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A naplózáshoz használt információk elsődleges forrásai az alábbiak lehetnek:

- A felhasználók hitelesítését kezelő biztonsági rendszer.
- A felhasználók tevékenységét rögzítő nyomkövetési naplók.
- Az azonosítható és azonosíthatatlan hálózati kéréseket nyomon követő biztonsági naplók.

A naplóadatok formátumára és tárolásuk módjára szabályozási követelmények vonatkozhatnak. Előfordulhat például, hogy az adatok semmilyen módon nem törölhetők. (Az eredeti formátumukban kell rögzíteni azokat.) Az adatokat tároló adattárt védeni kell az illetéktelen hozzáférés ellen.

### <a name="analyzing-audit-data"></a>A naplóadatok elemzése

Az elemzőknek el kell tudniuk érni a nyers adatok egészét, azok eredeti formájában. Az általános naplózási jelentések készítésének igénye mellett az adatok elemzésére szolgáló eszközök valószínűleg specializáltak lesznek, és a rendszeren kívül vannak elhelyezve.

## <a name="usage-monitoring"></a>A használat monitorozása

A használat monitorozása az alkalmazás funkcióinak és összetevőinek használatát követi nyomon. Az operátor az alábbiakat teheti az összegyűjtött adatok használatával:

- A gyakran használt funkcióknak és a rendszer potenciálisan kritikus pontjainak az azonosítása. A nagy forgalmú elemek működése javítható funkcionális particionálással vagy akár replikációval a terhelés egyenletesebb elosztása érdekében. Az operátor ezen információk használatával megállapíthatja, hogy melyek azok ritkán használt funkciók, amelyeket érdemes lehet kivezetni vagy a rendszer egy jövőbeli verziójában lecserélni.

- Információk beszerzése a rendszer működési eseményeivel kapcsolatban a normál használat során. Például egy e-kereskedelmi webhely a tranzakciók számával és az azokért felelős ügyfelek mennyiségével kapcsolatos statisztikai adatokat rögzíthet. Ezek az adatok a kapacitástervezéshez használhatók az ügyfelek számának növekedésével.

- A rendszer teljesítményével vagy funkcióival kapcsolatos felhasználói elégedettség észlelése (lehetőleg közvetetten). HA például egy e-kereskedelmi rendszer ügyfelei rendszeresen, nagy számban hagyják félbe a vásárlást, annak oka a fizetési funkcióval kapcsolatos probléma lehet.

- Számlázási információk létrehozása. A kereskedelmi alkalmazások és a több-bérlős szolgáltatások díjakat számíthatnak fel az ügyfeleknek az általuk használt erőforrásokért.

- Kvóták érvényesítése. Ha egy több-bérlős rendszer valamelyik felhasználója meghaladja a feldolgozási időre vagy erőforrás-használatra vonatkozó, kifizetett kvótát egy adott időszak során, a hozzáférése korlátozható vagy a feldolgozási teljesítmény szabályozható.

### <a name="requirements-for-usage-monitoring"></a>A használat monitorozására vonatkozó követelmények

A rendszerhasználat vizsgálatához az operátornak általában az alábbiakat magukban foglaló információkat kell látnia:

- Az egyes alrendszerek által feldolgozott és az egyes erőforrásokhoz továbbított kérések száma.
- Az egyes felhasználók által végzett munka.
- Az egyes felhasználók által foglalt adattárterület.
- Az egyes felhasználók által elért erőforrások.

Az operátornak diagramokat is létre kell tudnia hozni. A grafikonokon például megjeleníthetők a leginkább erőforrás-igényes felhasználók, vagy a leggyakrabban elért erőforrások vagy rendszerfunkciók.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A használat monitorozása viszonylag magas szinten végezhető. Képes feljegyezni az egyes kérések kezdési és befejezési időpontját és a kérés jellegét (olvasás, írás stb., az adott erőforrástól függően). Ezek az információk az alábbi módokon szerezhetők be:

- Felhasználói tevékenység nyomon követése.
- Az egyes erőforrások használatát mérő teljesítményszámlálók rögzítése.
- Az egyes felhasználók erőforrás-felhasználásának monitorozása.

Mérési okokból azt is tudnia kell azonosítani, hogy az egyes műveleteket mely felhasználók hajtják végre, és hogy ezek a műveletek mely erőforrásokat használják. A gyűjtött információknak megfelelően részletezettnek kell lennie a pontos számlázáshoz.

## <a name="issue-tracking"></a>Problémakövetés

Az ügyfelek és egyéb felhasználók problémákat jelenthetnek, ha váratlan esemény vagy működés következik be a rendszerben. A problémakövetés az ilyen problémák kezelésével foglalkozik, a problémákat a rendszerben rejlő alapvető hibák elhárítására irányuló törekvésekhez társítva, valamint tájékoztatja a felhasználókat a lehetséges megoldásokkal kapcsolatban.

### <a name="requirements-for-issue-tracking"></a>A problémakövetésre vonatkozó követelmények

Az operátorok gyakran külön rendszer használatával végzik a problémakövetést, amelyeken rögzíthetik és jelenthetik a felhasználók által jelentett problémák részleteit. Ezek a részletek a felhasználók által megkísérelt feladatokat, a problémák tüneteit, az események sorozatát, valamint a küldött hiba- és figyelmeztető üzeneteket foglalhatják például magukban.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A problémakövetési adatok kezdeti forrása a felhasználó, aki a problémát először jelentette. A felhasználó további adatokat is biztosíthat, például:

- Összeomlási memóriakép (ha az alkalmazás tartalmaz a felhasználó asztali gépén futó összetevőt).
- Képernyő-pillanatfelvétel.
- A hiba bekövetkezésének dátuma és időpontja az egyéb környezeti adatokkal, például a felhasználó tartózkodási helyével együtt.

Ezek az adatok felhasználhatók a hibakeresési tevékenység során, valamint a segítségükkel összeállítható a fejlesztendő funkciók listája a szoftver jövőbeli kiadásaihoz.

### <a name="analyzing-issue-tracking-data"></a>A problémakövetési adatok elemzése

Előfordulhat, hogy több felhasználó is jelenti ugyanazt a problémát. A problémakövetési rendszernek társítania kell az azonos jelentéseket.

A hibakeresési tevékenység előrehaladását rögzíteni kell az egyes hibajelentésekben. Ha a probléma elhárítása megtörtént, az ügyfél tájékoztatható a megoldásról.

Ha a felhasználó egy olyan problémát jelent, amelynek már létezik megoldása a problémakövetési rendszerben, az operátornak tudnia kell azonnal tájékoztatni a felhasználót a megoldásról.

## <a name="tracing-operations-and-debugging-software-releases"></a>A műveletek nyomon követése, és a szoftverkiadások hibáinak elhárítása

Amikor egy felhasználó problémát jelent, a felhasználó gyakran csak a saját műveleteire kifejtett közvetlen hatásnak van tudatában. A felhasználó csak az általa tapasztalt működés eredményét tudja jelenteni a rendszer karbantartásáért felelős operátor számára. Ezek a tapasztalatok rendszerint csak a látható tünetei egy vagy több alapvető problémáknak. Sok esetben egy elemzőnek alaposan át kell tekintenie a mögöttes műveletek kronológiáját a probléma kiváltó okának meghatározásához. Ezt az eljárást *a kiváltó okok elemzésének* nevezik.

> [!NOTE]
> A kiváltó okok elemzése feltárhatja az alkalmazás tervezési hiányosságait. Ilyen esetekben át lehet dolgozni az érintett elemeket, és egy későbbi kiadás részeként telepíteni lehet azokat. Ez a folyamat alapos tervezést igényel, és a frissített összetevőket gondosan monitorozni kell.

### <a name="requirements-for-tracing-and-debugging"></a>A nyomkövetésre és hibakeresésre vonatkozó követelmények

A váratlan események és egyéb problémák követéséhez létfontosságú, hogy a monitorozási adatok elegendő információt biztosítsanak az elemző számára a problémák eredetének visszakövetéséhez és a bekövetkezett események sorának rekonstruálásához. Elegendő információnak kell rendelkezésre állnia ahhoz, hogy az elemző diagnosztizálhassa a problémák kiváltó okait. A fejlesztők ezután elvégezhetik a szükséges módosításokat a problémák újbóli bekövetkezésének a megakadályozásához.

### <a name="data-sources-instrumentation-and-data-collection-requirements"></a>Adatforrások, rendszerállapot-adatok és adatgyűjtési követelmények

A hibaelhárításnak része lehet az egyes műveletek keretében meghívott összes metódus (és azok paramétereinek) nyomon követése, így felépíthető egy fa, amely a rendszeren végighaladó logikai folyamatot ábrázolja, amikor egy ügyfél egy adott kérést kezdeményez. A rendszer által a folyamat eredményeként létrehozott kivételeket és figyelmeztetéseket rögzíteni és naplózni kell.

A hibakeresés támogatása érdekében a rendszer hurkokat biztosíthat, amelyek segítségével az operátor rögzítheti a rendszer kritikus pontjainak állapotadatait. Másik megoldásként a rendszer biztosíthat információkat minden lépésről, miközben a kiválasztott műveletek zajlanak. Az ilyen részletességi szintű adatok rögzítése további terhelést jelenthet a rendszer számára, ezért ideiglenes folyamatnak kell lennie. Az operátorok főleg akkor alkalmazzák ezt a folyamatot, amikor rendkívül szokatlan események nehezen megismételhető sorozata következik be, vagy ha a rendszerben újonnan bevezetett egy vagy több elemet gondosan monitorozni kell annak biztosításához, hogy az elemek az elvárt módon működnek.

## <a name="the-monitoring-and-diagnostics-pipeline"></a>A monitorozási és diagnosztikai folyamat

A kiterjedt, elosztott rendszerek megfigyelése meglehetősen nehéz. Az előző szakaszban leírt forgatókönyveket nem feltétlenül egymástól elkülönítve kell alkalmazni. Az egyes helyzetekhez szükséges monitorozási és diagnosztikai adatok vélhetően jelentős átfedésben lesznek, jóllehet előfordulhat, hogy az adatokat különböző módon kell feldolgozni és megjeleníteni. Ebből kifolyólag a monitorozást és a diagnosztikát holisztikus módon kell szemlélni.

A teljes monitorozási és diagnosztikai folyamatot egy folyamatként érdemes elképzelni, amely az 1. ábrán látható szakaszokat foglalja magában.

![A monitorozási és diagnosztikai folyamat szakaszai](./images/monitoring/Pipeline.png)

*1. ábra A monitorozási és diagnosztikai folyamat szakaszai.*

Az 1. ábra rámutat arra, hogy a monitorozási és diagnosztikai adatok számos különféle adatforrásból származhatnak. A rendszerállapot-megfigyelési és az adatgyűjtési szakasz feladata a rögzítendő adatok forrásainak azonosítása, a rögzítendő adatoknak, a rögzítés módjának és az adatok könnyű vizsgálhatóságot lehetővé tevő formátumának meghatározása. Az elemzés/diagnosztika szakasz a nyers adatokat felhasználva hasznos adatokat hoz létre, amelyek segítségével az operátor meghatározhatja a rendszer állapotát. Az operátor ezen információk alapján döntéseket hozhat a lehetséges intézkedésekre vonatkozóan, majd az eredményeket visszaküldheti a rendszerállapot-figyelési és az adatgyűjtési szakaszba. A megjelenítési/riasztási szakasz hasznavehető képet biztosít a rendszer állapotáról. Különféle irányítópultokon közel valós idejű információkat jeleníthet meg. Emellett jelentések, diagramok és grafikonok létrehozásával a hosszú távú trendek azonosítását segítő adatok előzménynézetét biztosíthatja. Ha az adatok azt jelzik, hogy egy KPI valószínűleg meghaladja majd az elfogadható határértékeket, ez a szakasz riasztást is küldhet az operátornak. Bizonyos esetekben a riasztással automatikus folyamat is indítható, amely korrekciós művelet, például automatikus skálázás végrehajtására tesz kísérletet.

Vegye figyelembe, hogy a lépéseket egy összefüggő folyamatot alkotnak, amelyben a szakaszok egymással párhuzamosan zajlanak. Ideális esetben a szakaszok dinamikusan konfigurálhatók. Bizonyos pontokon, különösen ha a rendszer újonnan lett üzembe helyezve vagy problémákba ütközik, nagyobb gyakorisággal lehet szükség többféle adat gyűjtésére. Máskor azonban vissza kell tudni állni csak a lényeges információk alapszintű gyűjtésére a rendszer megfelelő működésének ellenőrzéséhez.

Ezenfelül a teljes monitorozási folyamatot egy folyamatosan működő megoldásnak kell tekinteni, amely finomhangolásra és fejlesztésre szorul a visszajelzések alapján. Előfordulhat például, hogy induláskor esetleg rengeteg tényezőt mér a rendszer állapotának meghatározásához. Idővel az elemzés révén finomítható a működés, a nem releváns mérések elvethetők, és pontosabban összpontosíthat a valóban szükséges adatokra, miközben a háttérzaj minimalizálható.

## <a name="sources-of-monitoring-and-diagnostic-data"></a>A monitorozási és diagnosztikai adatok forrásai

A monitorozási folyamat által használt által információk számos különböző forrásból származhatnak, amint az az 1. ábrán is látható. Az alkalmazás szintjén az információk a rendszer kódjába ágyazott nyomkövetési naplókból származnak. A fejlesztőknek érdemes egy standard megközelítést alkalmazni a vezérlési folyam nyomon követésére a kódban. A metódusokba való belépéskor például kibocsátható egy nyomkövetési üzenet, amely megadja a metódus nevét, az aktuális időpontot, az egyes paraméterek értékét és minden egyéb releváns információt. A be- és kilépési idők rögzítése szintén hasznosnak bizonyulhat.

Az összes kivételt és figyelmeztetést naplózni kell, és gondoskodni kell a beágyazott kivételek és figyelmeztetések teljes nyomkövetésének a megőrzéséről. Ideális esetben a kódot futtató felhasználót azonosító adatokat is rögzíteni kell, a tevékenységek korrelálására szolgáló adatokkal egyetemben (a kérések nyomon követésére, miközben áthaladnak a rendszeren). Emellett naplózni kell az erőforrások, például az üzenetsorok, adatbázisok, fájlok és egyéb függő szolgáltatások elérésére tett kísérleteket is. Ezek az információk mérési és naplózási célokra használhatók.

Számos alkalmazás használ kódtárakat és keretrendszereket az általános feladatok végrehajtására, például az adattárak eléréséhez vagy a hálózaton keresztüli kommunikációhoz. Ezek a keretrendszerek konfigurálhatók saját nyomkövetési üzenetek és nyers diagnosztikai információk biztosítására, ilyen például a tranzakciók sebessége vagy az adatátvitel sikeressége és sikertelensége.

> [!NOTE]
> Számos modern keretrendszer automatikusan közzéteszi a teljesítményt és a nyomkövetési eseményeket. Ezen információk rögzítéséhez csupán eszközt kell biztosítani a lekérésükhöz, és olyan helyen kell tárolni őket, ahol feldolgozhatók és elemezhetők.

Az alkalmazást futtató operációs rendszer olyan alsó szintű rendszerinformációk forrásaként szolgálhat, mint az I/O-sebességet, a memória- és a processzorhasználatot jelző teljesítményszámlálók. Az operációs rendszer hibái (például egy fájl megfelelő megnyitásának sikertelensége) szintén jelenthetők.

A rendszert futtató mögöttes infrastruktúrát és összetevőket is számításba kell venni. A virtuális gépek, virtuális hálózatok és tárolószolgáltatások mind szolgálhatnak fontos, infrastruktúraszintű teljesítményszámlálók és egyéb diagnosztikai adatok forrásául.

Ha az alkalmazás egyéb külső szolgáltatásokat, például webkiszolgálót vagy adatbázis-kezelési rendszert is használ, ezek a szolgáltatások saját nyomkövetési adatokat, naplókat és teljesítményszámlálókat tehetnek közzé. Ilyenek például az SQL Server dinamikus felügyeleti nézetei az SQL Server-adatbázisra irányulóan végrehajtott műveletek nyomon követésére, és az IIS nyomkövetési naplói a webkiszolgálóhoz intézett kérések rögzítésére.

A rendszer összetevőinek módosításával és az újabb verziók üzembe helyezésével fontos, hogy a problémák, események és mérőszámok az egyes verziókhoz rendelhetők legyenek. Ezeket az információkat a kiadási folyamathoz kell kötni, hogy az összetevők adott verziójával kapcsolatos problémák gyorsan felderíthetők és javíthatók legyenek.

Biztonsági problémák a rendszer bármelyik pontján felléphetnek. Egy felhasználó például megpróbálhat érvénytelen felhasználói azonosítóval vagy jelszóval bejelentkezni. Egy hitelesített felhasználó megpróbálhat jogosulatlan hozzáférést szerezni egy erőforráshoz. Vagy egy felhasználó megpróbálhat egy érvénytelen vagy elavult kulcsot megadni a titkosított adatok eléréséhez. A sikeres és sikertelen kérések biztonsággal kapcsolatos adatait mindig naplózni kell.

A [Rendszerállapot-figyelés az alkalmazásban](#instrumenting-an-application) szakasz további útmutatást nyújt azon információkkal kapcsolatban, amelyeket rögzíteni kell. Az információk gyűjtésére azonban többféle stratégia használható:

- **Alkalmazás-/rendszermonitorozás**. Ez a stratégia az alkalmazáson, alkalmazás-keretrendszereken, operációs rendszereken és infrastruktúrán belüli forrásokat használ. Az alkalmazás kódja is létrehozhat saját monitorozási adatokat az ügyfélkérések életciklusának fontosabb pontjain. Az alkalmazás nyomkövetési utasításokat is tartalmazhat, amelyek szelektíven engedélyezhetők vagy tilthatók le a körülményeknek megfelelően. Diagnosztika is beszúrható dinamikusan diagnosztikai keretrendszer használatával. Ezek a keretrendszerek általában olyan beépülő modulokat biztosítanak, amelyek a kód különböző rendszerállapot-figyelési pontjaihoz csatlakozva rögzítik a pontok nyomkövetési adatait.
  
    Emellett a kód és/vagy a mögöttes infrastruktúra is létrehozhat eseményeket a kritikus pontokon. Az események figyelésére konfigurált monitorozási ügynökök rögzíthetik az eseményadatokat.

- **Valódi felhasználómonitorozás**. Ez a megközelítés a felhasználó és az alkalmazás közötti interakciót rögzíti, és az egyes kérések és válaszok útját figyeli. Ezek az információk két célra alkalmazhatók: az egyes felhasználók általi használat mérésére, valamint annak meghatározására, hogy a felhasználók megfelelő szolgáltatásminőséget kapnak-e (például gyors válaszidők, alacsony késés és minimális számú hiba). A rögzített adatok használatával azonosíthatók a problémás területek, ahol a hibák leggyakrabban fellépnek. Az adatok alapján azonosíthatók azok az elemek is, ahol a rendszer lelassul, vélhetően az alkalmazás forgalmas pontjai vagy a szűk keresztmetszet más formái miatt. Ha körültekintően alkalmazza ezt a megközelítést, a felhasználóknak az alkalmazásban követett útja esetleg rekonstruálható lesz hibakeresési és tesztelési célokra.
  
  > [!IMPORTANT]
  > A valódi felhasználók monitorozásával rögzített adatokat rendkívül kényes adatként kell kezelni, mivel bizalmas információkat is tartalmazhatnak. Ha menti a rögzített adatokat, biztonságosan tárolja azokat. Ha az adatokat a teljesítmény monitorázáshoz vagy hibakereséshez szeretné használni, először távolítsa el a személyes azonosításra alkalmas adatok mindegyikét.

- **Szintetikus felhasználómonitorozás**. Ebben a megközelítésben saját tesztügyfelet ír, amely szimulálja a felhasználókat, és egy konfigurálható, de általános műveletsort hajt végre. A tesztügyfél teljesítményének nyomon követésével meghatározhatja a rendszer állapotát. A tesztügyfél több példányát egy terheléstesztelési művelet részeként használva megállapítható, hogyan reagál a rendszer a magas terhelés alatt, és milyen monitorozási adatok keletkeznek ilyen körülmények közt.
  
  > [!NOTE]
  > A valódi és szintetikus felhasználómonitorozás olyan kód alkalmazásával valósítható meg, amely a metódushívásokat és az alkalmazás egyéb kritikus részeit követi nyomon és látja el időadatokkal.

- **Profilkészítés**. Ez a megközelítés elsősorban az alkalmazásteljesítmény monitorozását és javítását célozza. Ahelyett, hogy a valódi és szintetikus felhasználómonitorozás funkcionális szintjén működne, alacsonyabb szintű információkat rögzít az alkalmazás futása során. A profilkészítés az alkalmazások végrehajtási állapotának rendszeres mintavételezésével (az alkalmazás által az adott pillanatban futtatott kódrészlet meghatározásával) valósítható meg. Olyan rendszerállapot-figyelést is alkalmazhat, amely mintavételezőket szúr be a kód kritikus elágazásain (például a metódushívások indításánál és befejezésénél), és rögzíti, hogy mely metódusok lettek meghívva, mikor, és mennyi ideig tartottak az egyes hívások. Az adatok elemzésével aztán meghatározható, hogy az alkalmazás mely részei okozhatnak teljesítményproblémákat.

- **Végpont-monitorozás**. Ez a módszer egy vagy több diagnosztikai végpontot alkalmaz, amelyeket az alkalmazás kifejezetten a monitorozás lehető tétele céljából tesz elérhetővé. A végpontok egy utat biztosítanak az alkalmazás kódjába, és információkat adhatnak vissza a rendszer állapotával kapcsolatban. Különböző végpontok a működés különböző aspektusaira fókuszálhatnak. Írhat saját diagnosztikai ügyfelet, amely rendszeresen kéréseket küld ezekre a végpontokra, és feldolgozza a válaszokat. További információkért lásd: a [állapot végponti Monitorozását végző minta](../patterns/health-endpoint-monitoring.md).

A maximális lefedettség érdekében érdemes ezeknek a módszereknek valamilyen kombinációját alkalmaznia.

## <a name="instrumenting-an-application"></a>Rendszerállapot-figyelés az alkalmazásban

A rendszerállapot-figyelés a monitorozási folyamat kritikus részét képezi. A rendszer teljesítményével és állapotával kapcsolatban csak úgy hozhat megalapozott döntéseket, ha előbb rögzíti azokat az adatokat, amelyek lehetővé teszik ezen döntések meghozatalát. A rendszerállapot-figyelés segítségével gyűjtött információknak elegendőnek kell lenniük a teljesítmény értékeléséhez, a problémák diagnosztizálásához és a döntések meghozatalához anélkül, hogy be kellene jelentkezni egy távoli éles kiszolgálóra a nyomkövetés (és a hibakeresés) manuális végrehajtásához. A rendszerállapot-adatok általában mérőszámokból és a nyomkövetési naplókba írt információkból állnak.

A nyomkövetési napló tartalmát képezhetik az alkalmazás által írt szöveges adatok vagy a nyomkövetési események eredményeként keletkező bináris adatok (ha az alkalmazás Windows esemény-nyomkövetést (ETW) alkalmaz). Ezeket az adatokat az infrastruktúra részein, például a webkiszolgálón létrejövő eseményeket rögzítő rendszernaplók is létrehozhatják. A szöveges naplóüzenetek gyakran a felhasználók számára olvashatók, azonban olyan formátumban kell írni őket, amely lehetővé teszi az egyszerű elemzésüket is az automatikus rendszerek számára.

A naplókat kategorizálni is kell. Ne írjon minden nyomkövetési adatot ugyanabba a naplóba, hanem külön naplókba rögzítse a rendszer különböző működési területeinek nyomkövetési eredményeit. Így gyorsan szűrheti majd a naplóüzeneteket, ha azokat a megfelelő naplóból olvassa be, és nem egyetlen hosszú fájlt kell feldolgoznia. Ne írjon különböző biztonsági követelményekkel rendelkező információt (például naplózási információkat és hibakeresési adatokat) ugyanazon naplóba.

> [!NOTE]
> A napló lehet egy fájl a fájlrendszerben, vagy menthető más formátumba is, például blobként a Blob Storage-tárolóban. A naplóadatok tárolhatók strukturáltabb módon is, például egy tábla soraiként.

A mérőszámok általában a rendszer egy adott aspektusára vagy erőforrására és egy adott időpontra vonatkozó mérések vagy darabszámok, amelyek egy vagy több hozzárendelt címkével vagy dimenzióval rendelkeznek (ezeket néha *mintának* is szokás nevezni). A mérőszámok egyes példányai önmagukban általában nem használhatók, hanem hosszabb időtartamra vonatkozóan kell rögzíteni őket. A legfontosabb azt figyelembe venni, hogy pontosan mely mérőszámokat és milyen gyakorisággal kell rögzíteni. A mérőszámadatok túl gyakori gyűjtése jelentős további terhelést jelent a rendszer számára, míg a túl ritka gyűjtés esetén lemaradhat az egyes jelentős események keletkezését okozó körülményekről. A szempontok mérőszámonként eltérőek lehetnek. A processzorhasználat egy kiszolgálón például egyik másodpercről a másikra változhat, de a magas kihasználtság csak akkor válik problémává, ha hosszan, több percen keresztül tart.

### <a name="information-for-correlating-data"></a>Információk az adatok korrelálásához

Egyszerűen monitorozhatja az egyes rendszerszintű teljesítményszámlálókat, rögzítheti az erőforrások mérőszámait, és szerezhet be alkalmazás-nyomkövetési információkat a különböző naplófájlokból. A monitorozás bizonyos formái esetén azonban szükség van egy elemzési és diagnosztikai szakaszra a monitorozási folyamatban a különféle forrásokból származó adatok korrelálásához. Ezek az adatok különféle formákat ölthetnek a nyers adatokban, és az elemzési folyamatot el kell látni megfelelő mennyiségű rendszerállapot-adattal, hogy képes legyen leképezni ezeket a különféle formákat. Tegyük fel például, hogy az alkalmazás-keretrendszer szintjén az egyes feladatokat egy szálazonosító azonosítja. Az alkalmazáson belül ugyanazon feladat az azt végrehajtó felhasználó felhasználói azonosítójához társítható.

Emellett nem is valószínű, hogy 1:1 leképezés hozható létre a szálak és a felhasználói kérések között, mivel az aszinkron műveletek ugyanannak a szálnak a használatával több felhasználó nevében is végezhetnek műveleteket. A dolgot tovább bonyolítja, hogy egyetlen kérést több szál is kezelhet, ahogy a végrehajtás végighalad a rendszeren. Amennyiben lehetséges, az egyes kérésekhez egy egyedi tevékenységazonosítót társítson, amelyet a végrehajtás propagál a rendszeren a kérés környezetének részeként. (A tevékenységazonosítóknak létrehozásának és nyomkövetési adatokba való foglalásának módszere a nyomkövetési adatok rögzítésére szolgáló technológiától függ.)

Minden monitorozási adatot ugyanúgy kell időbélyeggel ellátni. A konzisztencia érdekében minden dátumot és időpontot az egyezményes világidő (UTC) használatával rögzítsen. Ez megkönnyíti az eseménysorozatok nyomon követését.

> [!NOTE]
> Előfordulhat, hogy a különböző időzónákban és hálózatokon működő számítógépek nincsenek szinkronizálva. Ne támaszkodjon kizárólag az időbélyegekre a több számítógépet is érintő rendszerállapot-adatok korrelálásához.

### <a name="information-to-include-in-the-instrumentation-data"></a>A rendszerállapot-adatokba foglalandó információk

Amikor mérlegeli, hogy milyen információkat kell gyűjtenie, vegye figyelembe az alábbi szempontokat:

- Gondoskodjon róla, hogy a nyomkövetési események által rögzített információk gépi és emberi olvasásra egyaránt alkalmasak legyenek. Alkalmazzon jól meghatározott sémákat az ilyen információkhoz, hogy megkönnyítse a naplóadatok automatizált feldolgozását a különféle rendszereken, valamint hogy biztosítsa a konzisztenciát a naplókat használó üzemeltetési és fejlesztési személyzet számára. Az információkat egészítse ki környezeti adatokkal, például a telepítési környezetre, a folyamatot futtató gépre, a folyamat részleteire és a hívási veremre vonatkozó adatokkal.

- A profilkészítést csak abban az esetben engedélyezze, ha valóban szükséges, mivel jelentős többletterhelésnek teheti ki a rendszert. A rendszerállapot-adatokat használó profilkészítés minden egyes alkalommal rögzíti az eseményeket (például a metódushívásokat) azok előfordulásakor, míg a mintavételezés csak a kiválasztott eseményeket rögzíti. A kijelölt lehet időalapú (után minden *n* másodperc), vagy gyakorisága-alapú (miután minden *n* kérések). Ha túl gyakran történnek események, a rendszerállapot-adatokat használó profilkészítés túl nagy terhet jelenthet, és negatív hatással lehet az általános teljesítményre. Ebben az esetben a mintavételezéses módszer használata előnyösebb lehet. Ha azonban az események gyakorisága alacsony, előfordulhat, hogy a mintavételezés lemarad róluk. Ebben az esetben a rendszerállapot-figyelés bizonyulhat a hatékonyabb megközelítésnek.

- Biztosítson elegendő kontextust, amely alapján a fejlesztők és rendszergazdák megállapíthatják az egyes kérések forrását. A kontextus tartalmazhat például valamilyen tevékenységazonosítót, amely azonosítja a kérés egy adott példányát. Emellett tartalmazhat olyan információkat is, amelyek használatával a tevékenység korrelálható az elvégzett számítási munkával és a használt erőforrásokkal. Vegye figyelembe, hogy a munka esetleg túlnyúlik a folyamatok és a gépek határain. Mérés esetén a kontextusnak tartalmaznia kell (közvetlenül vagy más korrelált információkon keresztül közvetve) a kérés létrehozását kiváltó felhasználóra vonatkozó hivatkozást is. A kontextus értékes információkat biztosít az alkalmazásnak a monitorozási adatok rögzítésének időpontjában érvényes állapotára vonatkozóan.

- Minden kérést rögzíteni kell, a kérések kezdeményezésének helyével vagy régiójával együtt. Ezen információk segíthetnek annak meghatározásában, hogy vannak-e helyspecifikus kritikus pontok a rendszerben. Az információk hasznosak lehetnek annak meghatározásához is, hogy érdemes-e az alkalmazást vagy az általa használt adatokat újraparticionálni.

- A kivételek adatait szintén gondosan rögzíteni kell. A kritikus fontosságú hibakeresési információk gyakran elvesznek a kivételek nem megfelelő kezelésének eredményeként. Az alkalmazás által jelentett kivételek minden adatát rögzíteni kell, beleértve a belső kivételekre vonatkozó és az egyéb környezeti információkat is. Ha lehetséges, a hívási vermet is rögzíteni kell.

- Az alkalmazás különböző elemei által rögzített adatoknak konzisztensnek kell lenniük, ez ugyanis segíthet az események elemzésében, és azoknak a felhasználói kérésekkel való korrelálásában. Vegye fontolóra egy átfogó és konfigurálható naplózási csomag alkalmazását az információk gyűjtésére, és inkább nem számítson arra, hogy a fejlesztők ugyanazt a megközelítést alkalmazzák majd a rendszer különböző részeinek megvalósítása során. Gyűjtse a kulcsfontosságú teljesítményszámlálók adatait, például a végrehajtott I/O-műveletek mennyiségét, a hálózathasználatot, a kérések számát, a memóriahasználatot és a processzorhasználatot. Egyes infrastruktúraszolgáltatások saját teljesítményszámlálókat is biztosíthatnak, például az adatbázisokkal létesített kapcsolatok számát, a tranzakciók végrehajtási sebességét, valamint a sikeres vagy sikertelen tranzakciók számát. Emellett az alkalmazások is meghatározhatnak saját teljesítményszámlálókat.

- Naplózza a külső szolgáltatásokra, például adatbázisrendszerekre, webszolgáltatásokra vagy az infrastruktúra részét képező egyéb rendszerszintű szolgáltatásokra irányuló összes hívást. Rögzítse az egyes hívások végrehajtásához szükséges időkkel, valamint a hívások sikerességével vagy sikertelenségével kapcsolatos információkat is. Amennyiben lehetséges, rögzítse az összes újrapróbálkozási kísérletet és hibát is az előforduló átmeneti hibák esetén.

### <a name="ensuring-compatibility-with-telemetry-systems"></a>A telemetriarendszerekkel való kompatibilitás biztosítása

Sok esetben a rendszerállapot-figyelés által létrehozott információk események sorozataként jönnek létre, amelyeket egy telemetriarendszer vesz át feldolgozásra és elemzésre. A telemetriarendszerek általában mindenféle alkalmazástól vagy topológiától függetlenek, de elvárják, hogy az információk egy specifikus, általában valamilyen séma alapján meghatározott formátumot kövessenek. A séma lényegében egy megállapodást jelent, amely meghatározza azon adatmezőket és -típusokat, amelyeket a telemetriarendszer fogadni képes. A sémát érdemes általánosítani, hogy a különféle platformokról és eszközökről érkező adatok mindegyikét kezelni tudja.

Az általános sémának tartalmaznia kell az összes rendszerállapot-figyelési eseményben megtalálható lévő mezőket, például az esemény nevét, az esemény időpontját, a küldő IP-címét, valamint a más eseményekkel való korreláláshoz szükséges adatokat (például a felhasználóazonosítót, eszközazonosítót és alkalmazásazonosítót). Ne feledje, hogy a legkülönfélébb eszközök hozhatnak létre eseményeket, ezért a sémának nem szabad függenie az eszköztípustól. Emellett a különböző eszközök ugyanarra az alkalmazásra vonatkozó eseményeket hozhatnak létre, ezért az alkalmazásnak támogatnia kell a barangolást vagy a különböző eszközök közötti terjesztés valamilyen más formáját.

A séma tartalmazhat tartományi mezőket is, amelyek a különböző alkalmazásokban közös forgatókönyvre jellemzők. A mezők tartalmazhatnak a kivételekkel, az alkalmazásindítási és -bezárási eseményekkel, valamint a webszolgáltatás API-hívásainak sikerével és/vagy sikertelenségével kapcsolatos információk. Az ugyanazon tartományi mezőket használó alkalmazások mindegyikének ugyanazokat az eseményeket kell kibocsátania, ami lehetővé teszi a közös jelentések és elemzések létrehozását.

Végül a séma tartalmazhat egyéni mezőket is az alkalmazásspecifikus események adatainak a rögzítéséhez.

### <a name="best-practices-for-instrumenting-applications"></a>Ajánlott eljárások az alkalmazásokban zajló rendszerállapot-figyeléshez

Az alábbi lista a felhőben futó elosztott alkalmazás rendszerállapot-figyelésére vonatkozó ajánlott eljárásokat foglalja össze.

- A naplók legyenek könnyen olvashatók és elemezhetők. Alkalmazzon strukturált naplózást, ahol lehetséges. A naplóüzenetek legyenek tömörek és leíró jellegűek.

- Minden naplóban azonosítsa a forrást, és biztosítson környezeti és időadatokat az egyes bejegyzések írásakor.

- Használja ugyanazt az időzónát és formátumot minden időbélyeghez. Így könnyebben korrelálhatja az olyan műveletek eseményeit, amelyek különböző régiókban futó hardvereken és szolgáltatásokon futnak.

- Kategorizálja a naplókat, és írja az üzeneteket a megfelelő naplófájlba.

- Ne tegye közzé a rendszerrel kapcsolatos bizalmas adatokat és a felhasználók személyes adatait. A naplózás előtt tisztítsa meg az adatokat ezektől az információktól, azonban a releváns részleteket mindenképpen őrizze meg. Törölje például az azonosítót és a jelszót minden adatbázis-kapcsolati sztringből, a megmaradt információkat azonban írja a naplóba, így az elemzők megállapíthatják, hogy a rendszer a megfelelő adatbázist éri-e el. Naplózza az összes kritikus kivételt, azonban tegye lehetővé a rendszergazda számára a naplózás be-/kikapcsolását az alacsonyabb szintű kivételek és figyelmeztetések esetén. Rögzítse és naplózza az újrapróbálkozási logikával kapcsolatos összes információt is. Ezek az adatok hasznosnak bizonyulhatnak a rendszer átmeneti állapotának a monitorozása során.

- Kövesse a folyamatokon kívülre indított hívásokat, például a külső webszolgáltatásokra és adatbázisokra irányuló kéréseket.

- Ne keverje a különböző biztonsági szintű naplóüzeneteket ugyanabban a naplófájlban. Ne írja például a hibakeresési és a naplózási információkat ugyanabba a naplófájlba.

- A naplózási eseményeket kivéve, gondoskodjon róla, hogy az összes naplóhívás egyszeri művelet legyen, amely nem gátolja az üzleti tevékenységek végrehajtását. A naplózási események azért kivételesek, mivel üzleti szempontból kulcsfontosságúak, és az üzleti tevékenységek alapvető részének tekinthetők.

- Gondoskodjon róla, hogy a naplózás bővíthető legyen, és ne függjön közvetlenül semmilyen konkrét céltól. Például ahelyett, hogy a *System.Diagnostics.Trace* használatával írná az adatokat, definiáljon egy absztrakt felületet (például *ILogger*), amely naplózási metódusokat tesz elérhetővé, és amely bármilyen megfelelő módszerrel megvalósítható.

- Gondoskodjon róla, hogy a naplózás hibamentes legyen, és ne váltson ki sorozatosan egymásra épülő hibákat. A naplózásnak nem szabad kivételeket jelentenie.

- A rendszerállapot-figyelést egy folyamatosan zajló, iteratív folyamatként kell kezelni, és a naplókat rendszeresen át kell tekinteni, nem csupán probléma esetén.

## <a name="collecting-and-storing-data"></a>Az adatok gyűjtése és tárolása

A monitorozási folyamat adatgyűjtési szakaszának feladata a rendszerállapot-figyelés során létrehozott információk lekérése, az adatok formázása az elemzési/diagnosztikai szakaszban történő könnyebb feldolgozás céljából, valamint az átalakított adatok mentése egy megbízható tárolóba. Az elosztott rendszerek különböző részeiből gyűjtött rendszerállapot-adatok különféle helyeken és különböző formátumokban tárolhatók. Az alkalmazás kódja például nyomkövetési naplófájlokat és az alkalmazáseseményekhez kapcsolódó naplóadatokat hozhat elő, míg az alkalmazás által használt infrastruktúra kritikus pontjait monitorozó teljesítményszámlálók más technológiákkal rögzíthetők. Az alkalmazás által használt külső összetevők és szolgáltatások eltérő formátumokban biztosíthatják a rendszerállapot-adatokat külön profilelemzési fájlok, blobtárolók vagy akár egyéni adattárak használatával.

Az adatgyűjtés gyakran a rendszerállapot-adatokat létrehozó alkalmazástól függetlenül futtatható adatgyűjtési szolgáltatás használatával történik. A 2. ábrán erre az architektúrára látható egy példa, amely kiemeli a rendszerállapot-adatokat gyűjtő alrendszert.

![Példa a rendszerállapot-adatok gyűjtésére](./images/monitoring/TelemetryService.png)

*2. ábra Rendszerállapot-adatok gyűjtésére.*

Vegye figyelembe, hogy ez egy egyszerűsített ábra. Az adatgyűjtési szolgáltatás nem feltétlenül egyetlen folyamat, és akár különböző gépeken futó több összetevőből is állhat, amint azt a következő szakaszok ismertetik. Továbbá, ha bizonyos telemetriaadatok elemzését gyorsan kell elvégezni (forró elemzés, a dokumentum későbbi, a [Forró, meleg és hideg elemzés támogatása](#supporting-hot-warm-and-cold-analysis) című szakaszban leírtak szerint), az adatgyűjtési szolgáltatáson kívül működő helyi összetevők azonnal elvégezhetik az elemzési feladatokat. A 2. ábra ezt a helyzetet ábrázolja a kiválasztott eseményekre vonatkozóan. Az elemzést követően az eredmények közvetlenül a megjelenítési és riasztási alrendszerbe küldhetők. A meleg vagy hideg elemzésnek alávetett adatokat a tároló tárolja, amíg feldolgozásra várnak.

Az Azure-alkalmazások és -szolgáltatások esetében az Azure Diagnostics egyetlen lehetséges megoldást kínál az adatok rögzítésére. Az Azure Diagnostics az egyes számítási csomópontok alábbi forrásaiból gyűjti az adatokat, majd összesíti azokat, és feltölti az Azure Storage-ba:

- IIS-naplók
- sikertelen kéréseket tartalmazó IIS-naplók,
- Windows-eseménynaplók,
- Teljesítményszámlálók
- összeomlási memóriaképek,
- Azure Diagnostics-infrastruktúranaplók,  
- egyéni hibanaplók,
- .NET EventSource,
- jegyzékalapú ETW.

További információkért tekintse meg a cikket [Azure: Telemetria alapjai és hibaelhárítása](https://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx).

### <a name="strategies-for-collecting-instrumentation-data"></a>A rendszerállapot-adatok gyűjtésére vonatkozó stratégiák

Figyelembe véve a felhő rugalmas jellegét, és elkerülendő, hogy a telemetriaadatokat manuálisan kelljen lekérni a rendszer minden egyes csomópontjáról, gondoskodjon arról, hogy a rendszer egy központi helyre továbbítsa és ott konszolidálja az adatokat. A több adatközpontra is kiterjedő rendszerek esetén először hasznos lehet régiónként gyűjteni, konszolidálni és tárolni az adatokat, majd ezeket a regionális adatokat egyetlen központi rendszerbe összesíteni.

A sávszélesség-használat optimalizálása érdekében a kevésbé sürgős adatokat tömbökben, kötegekként is továbbíthatja. Az adatokat azonban nem szabad a végtelenségig késleltetni, különösen ha időérzékeny információkat tartalmaznak.

#### <a name="pulling-and-pushing-instrumentation-data"></a>*A rendszerállapot-adatok lekérése és leküldése*

A rendszerállapot-adatokat gyűjtő alrendszer aktívan lekérheti a rendszerállapot-adatokat a különféle naplókból és egyéb forrásokból az alkalmazás egyes példányaira vonatkozóan (*lekéréses modell*). Az alrendszer működhet passzív fogadóként is, amely azt várja, hogy az alkalmazás egyes példányait alkotó összetevők elküldjék az adatokat (*leküldéses modell*).

A lekéréses modell megvalósításának egyik módszere az alkalmazás egyes példányaival helyileg futó monitorozási ügynökök használata. A monitorozási ügynök egy külön folyamat, amely rendszeres időközönként beolvassa (lekéri) a helyi csomóponton gyűjtött telemetria-adatokat, és ezeket az információkat közvetlenül az alkalmazás összes példánya által közösen használt központi tárolóba írja. Az Azure Diagnostics ezt a mechanizmust valósítja meg. Az Azure webes vagy feldolgozói szerepköreinek minden példánya konfigurálható a helyben tárolt diagnosztikai és egyéb nyomkövetési adatok gyűjtésére. Az egyes példányok mellett futó monitorozási ügynökök átmásolják a meghatározott adatokat az Azure Storage-ba. A folyamattal kapcsolatban [a diagnosztikának az Azure Cloud Services és a Virtual Machines szolgáltatásban való engedélyezésével](/azure/cloud-services/cloud-services-dotnet-diagnostics) foglalkozó cikk biztosít további információt. Bizonyos elemek, például az IIS-naplók, az összeomlási memóriaképek és az egyéni hibanaplók írása blobtárolókba történik. A Windows-eseménynaplók, ETW-események és a teljesítményszámlálók adatait pedig táblatárolók rögzítik. A 3. ábra ezt a mechanizmust ábrázolja.

![Az információk monitorozási ügynök használatával való lekérését és megosztott tárolóba való írását bemutató ábra.](./images/monitoring/PullModel.png)

*3. ábra Monitorozási ügynök használatával kérje le az adatokat, és megosztott tárolóba való írását.*

> [!NOTE]
> A monitorozási ügynökök ideális megoldást jelentenek az adatforrásokból magától értetődően lekért rendszerállapot-adatok rögzítéséhez. Ilyen adatok például az SQL Server dinamikus felügyeleti nézeteiből származó adatok vagy az Azure Service Bus-üzenetsor hossza.

Az imént ismertetett megközelítés az egy helyen, korlátozott számú csomóponton futó, kisléptékű alkalmazás telemetriaadatainak tárolására is alkalmazható. Az összetett, nagy mértékben skálázható, globális, felhőalkalmazások azonban hatalmas mennyiségű adatot állíthatnak elő több száz webes és feldolgozói szerepkörből, adatbázis-szilánkból és egyéb szolgáltatásból. Az adatok ilyen mértékű áramlása könnyen túlterhelheti az egyetlen központi helyen rendelkezésre álló I/O-sávszélességet. A telemetriamegoldásnak ezért skálázhatónak kell lennie, nehogy szűk keresztmetszetet jelentsen a rendszer növekedése során. Ideális esetben a megoldásnak bizonyos fokú redundanciát is biztosítania kell, így csökkenthető a fontos monitorozási adatok (például a naplózási vagy számlázási adatok) elvesztésének kockázata a rendszer valamelyik részének meghibásodása esetén.

Ezen problémák megoldására megvalósítható a sorkezelés, amint az a 4. ábrán látható. Ebben az architektúrában a helyi monitorozási ügynök (ha megfelelően konfigurálható) vagy egy egyéni adatgyűjtési szolgáltatás (ha nem) az adatokat közzé teszi egy üzenetsorban. Egy aszinkron módon futó másik folyamat (a 4. ábrán bemutatott tárolóírási szolgáltatás) fogja az üzenetsorban lévő adatokat, és a megosztott tárolóba írja azokat. Az üzenetsorok megfelelő megoldást kínálnak az ilyen forgatókönyvekben, mivel a „legalább egyszeri” szemantikájuk révén biztosítják, hogy az üzenetsorba küldött adatok nem vesznek el a közzétételük után. A tárolóírási szolgáltatás egy külön feldolgozói szerepkör használatával valósítható meg.

![A rendszerállapot-adatok üzenetsorral való pufferelésének ábrája.](./images/monitoring/BufferedQueue.png)

*4. ábra Rendszerállapot-adatok pufferelése üzenetsorral.*

A helyi adatgyűjtési szolgáltatás közvetlenül a fogadásuk után hozzáadhatja az adatokat az üzenetsorhoz. Az üzenetsor pufferként szolgál, és így a tárolóírási szolgáltatás a saját tempójában kérheti le és írhatja az adatokat. Alapértelmezés szerint az üzenetsorok érkezési sorrend alapján működnek. Az üzenetek azonban rangsorolhatók is, így a gyorsan kezelendő adatokat tartalmazó üzenetek gyorsabban haladhatnak végig az üzenetsoron. További információkért lásd: a [elsőbbségi üzenetsor mintája](../patterns/priority-queue.md). Használhat más csatornákat (például Service Bus-témaköröket) is az adatok más célhelyekre való irányítására a szükséges elemzési módszertől függően.

A skálázhatóság érdekében a tárolóírási szolgáltatás több példányát is futtathatja. Nagy mennyiségű esemény esetén az adatokat egy eseményközpont segítségével irányíthatja különböző számítási erőforrásokhoz feldolgozásra és tárolásra.

#### <a name="consolidating-instrumentation-data"></a>*Rendszerállapot-adatok konszolidálása*

Az adatgyűjtési szolgáltatás által az egyes alkalmazáspéldányokról lekért rendszerállapot-adatok alapján az adott példány állapotára és teljesítményére vonatkozó helyi nézet hozható létre. A rendszer általános állapotának értékeléséhez az ilyen helyi nézetekben található adatok bizonyos szempontjait konszolidálni kell. Ez az adatok tárolása után tehető meg, bizonyos esetekben azonban már az adatgyűjtés során is megvalósítható. A megosztott tárolóba való közvetlen írás helyett a rendszerállapot-adatok áthaladhatnak egy külön adatkonszolidálási szolgáltatáson, amely egyesíti az adatokat, és egyfajta szűrő- és tisztítófolyamatként is működik. Az azonos korrelációs adatokat (például tevékenységazonosítót) tartalmazó rendszerállapot-adatok például egybeolvaszthatók. (Előfordulhat, hogy a felhasználó egy adott csomóponton elindít valamilyen üzleti műveletet, majd a csomópont meghibásodása esetén vagy terheléselosztási beállítások miatt a rendszer a műveletet áthelyezi egy másik csomópontra.) Ez a folyamat képes továbbá észlelni és eltávolítani a duplikált adatokat (ezeknek mindig fennáll az esélye, ha a telemetriaszolgáltatás üzenetsorok használatával küldi el a rendszerállapot-adatokat a tárolóba). Az 5. ábra erre a struktúrára mutat be egy példát.

![Példa szolgáltatás használatára a rendszerállapot-adatok konszolidálásához.](./images/monitoring/Consolidation.png)

*5. ábra Egy különálló szolgáltatás konszolidálására és tisztítására rendszerállapot-adatok használata.*

### <a name="storing-instrumentation-data"></a>Rendszerállapot-adatok tárolása

Az előző fejtegetések meglehetősen egyszerű képet festettek a rendszerállapot-adatok tárolásának módjáról. A valóságban azonban célszerű lehet a különböző típusú adatokat olyan technológiák használatával tárolni, amelyek a leginkább megfelelők az egyes típusok vélhető felhasználási módjának.

Például az Azure Blob Storage és Table Storage az elérésük szempontjából valamelyest hasonlóak. A használatukkal végrehajtható műveletek szempontjából azonban bizonyos korlátokkal rendelkeznek, és a bennük tárolt adatok részletessége is igen különböző. Ha további elemzési műveleteket kell végrehajtania, vagy teljes szöveges keresési képességekre van szüksége az adatok vonatkozásában, jobb megoldás lehet a különféle lekérdezés- és adathozzáférés-típusokra optimalizált képességeket biztosító adattárakat használni. Példa:

- A teljesítményszámlálók adatainak SQL-adatbázisban való tárolása lehetővé teszi az ad hoc elemzést.
- A nyomkövetési naplókat érdemesebb lehet az Azure Cosmos DB-ben tárolni.
- A biztonsággal kapcsolatos információk a HDFS-be írhatók.
- A teljes szöveges keresést igénylő információk az Elasticsearch használatával tárolhatók (ami a gazdag indexelésnek köszönhetően növelheti a keresések sebességét).

Megvalósíthat egy további szolgáltatást is, amely rendszeres időközönként lekéri az adatokat a megosztott tárolóból, a céljuknak megfelelően particionálja és szűri azokat, majd a megfelelő adattárakba írja őket, amint az a 6. ábrán látható. Egy másik megközelítés ezen funkció belefoglalása a konszolidálási és tisztítási folyamatba, és az adatoknak közvetlenül ezen tárolókba, nem pedig egy közbenső megosztott tárterületre való írása a lekérésüket követően. Mindegyik megközelítésnek vannak előnyei és hátrányai. Egy külön particionálási szolgáltatás megvalósítása csökkenti a konszolidálási és tisztítási szolgáltatás terhelését, és lehetővé teszi, hogy szükség esetén legalább a particionált adatok egy részét újra létre lehessen hozni (attól függően, hogy mekkora a megosztott tárolóban megőrzött adatok mennyisége). Ez azonban további erőforrásokat használ fel. Emellett elképzelhető, hogy a rendszerállapot-adatok később érkeznek meg az egyes alkalmazáspéldányokról, és később történik a gyakorlatban használható információkká való átalakításuk.

![Az adatok particionálása és tárolása](./images/monitoring/DataStorage.png)

*6. ábra Particionálási adatok igény szerinti elemzési és tárolási követelményeinek.*

Ugyanazok a rendszerállapot-adatok több célból is szükségesek lehetnek. A teljesítményszámlálók segítségével például megjeleníthető az idő múlásával tapasztalható rendszerteljesítmény előzménynézete. Ezeket az információkat más használati adatokkal egyesítve létrehozhatók az ügyfélre vonatkozó számlázási információk. Ilyen esetekben ugyanazokat az adatokat esetleg több célhelyre is továbbítani kell, például a számlázási információk hosszú távú tárolására szolgáló dokumentum-adatbázisba, és a komplex teljesítményelemzés kezelését lehetővé tevő többdimenziós tárolóba.

Azt is figyelembe kell venni, hogy mennyire sürgősen van szükség az adatokra. A riasztásokhoz szükséges információkat biztosító adatokat gyorsan kell elérni, ezért egy gyors adattárban, indexelve vagy strukturálva kell tárolni őket a riasztási rendszer által végrehajtott lekérdezések optimalizálásához. Egyes esetekben szükséges lehet, hogy az egyes csomópontokon az adatokat gyűjtő telemetriaszolgáltatás az adatokat helyben formázza és mentse, hogy a riasztási rendszer helyi példánya gyorsan értesíthesse a felhasználókat a problémákról. Ugyanezek az adatok elirányíthatók az előző ábrákon látható tárolóírási szolgáltatásnak, és központilag tárolhatók, ha más célokból is szükség lenne rájuk.

Az alaposabb elemzéshez, a jelentéskészítéshez és az előzménytrendek észleléséhez használt információk kevésbé sürgősek, és olyan módon tárolhatók, amely támogatja az adatbányászatot és az ad hoc lekérdezéseket. További információért tekintse meg a [Forró, meleg és hideg elemzés támogatása](#supporting-hot-warm-and-cold-analysis) című szakaszt a dokumentum későbbi részében.

#### <a name="log-rotation-and-data-retention"></a>*Naplóváltás és adatmegőrzés*

A rendszerállapot-figyelés jelentős mennyiségű adatot hozhat létre. Ezek az adatok több helyen tárolhatók, az egyes csomópontokon rögzített nyers naplófájloktól, profilelemzési fájloktól és egyéb információktól a megosztott tárolóban található konszolidált, megtisztított és particionált nézetekig. Néhány esetben az eredeti nyers forrásadatok eltávolíthatók a csomópontokról az adatok feldolgozását és továbbítását követően. Más esetekben azonban szükséges vagy egyszerűen csak hasznos lehet menteni a nyers adatokat. A hibakeresési célból létrehozott adatokat például jobb lehet nyers formájukban meghagyni, azonban a hibák javítását követően gyorsan elvethetők.

A teljesítményadatok általában hosszabb életűek, mivel így felhasználhatók a teljesítménytrendek észlelésére és a kapacitástervezéshez. Az ilyen adatok konszolidált nézeteit szokás a gyorsabb elérés érdekében határozott ideig online megőrizni. Ezt követően archiválhatók vagy elvethetők. A mérési és számlázási célból gyűjtött adatokat néha határozatlan ideig meg kell őrizni. Emellett szabályozási követelmények is előírhatják, hogy a naplózási és biztonsági célból gyűjtött információkat archiválni és menteni kell. Ezek az adatok bizalmasak is, és emiatt előfordulhat, hogy titkosítani vagy más módon védeni kell azokat az illetéktelen hozzáférés ellen. A felhasználók jelszavait és a személyazonossági csalások elkövetéséhez felhasználható egyéb adatait soha nem szabad rögzíteni. Ezeket az információkat el kell távolítani az adatokból a mentésük előtt.

#### <a name="down-sampling"></a>*A mintavételezés sebességének csökkentése*

Az előzményadatokat célszerű menteni a hosszú távú trendek felismerése érdekében. A régi adatok egészének való mentése helyett azonban csökkenthető az adatok mintavételezésének sebessége az adatok felbontásának és a tárolási költségeknek a csökkentéséhez. A percenkénti teljesítménymutatók mentése helyett konszolidálhatja az egy hónapnál régebbi adatokat egy órára lebontott nézet létrehozásához.

### <a name="best-practices-for-collecting-and-storing-logging-information"></a>Ajánlott eljárások a naplózási információk gyűjtéséhez és tárolásához

Az alábbi lista a naplózási információk gyűjtésére és tárolására vonatkozó ajánlott eljárásokat foglalja össze:

- A monitorozási ügynököt vagy az adatgyűjtési szolgáltatást futtassa folyamaton kívüli szolgáltatásként, és legyenek egyszerűen üzembe helyezhetők.

- A monitorozási ügynök vagy az adatgyűjtési szolgáltatás minden kimenetének géptől, operációs rendszertől és hálózati protokolltól független semleges formátummal kell rendelkeznie. Az ETL/ETW helyett például az adatok kibocsáthatók egy önleíró formátumban, amilyen a JSON, a MessagePack vagy a Protobuf. A szabványos formátumok használata lehető teszi a rendszer számára feldolgozási folyamatok létrehozását. Az adatokat az egyeztetett formátumban olvasó, átalakító és küldő összetevők könnyen integrálhatók.

- A monitorozási és adatgyűjtési folyamatnak hibamentesnek kell lennie, és nem válthat ki sorozatosan egymásra épülő hibákat.

- Az információk adatfogadóba való küldésének átmeneti hibája esetén a monitorozási ügynöknek vagy az adatgyűjtési szolgáltatásnak készen kell állni a telemetriaadatok újrarendezésére, hogy a legújabb információk küldése történjen meg először. (A monitorozási ügynök/adatgyűjtési szolgáltatás saját belátása szerint dönthet úgy, hogy a régebbi adatokat elveti, vagy helyben menti, és később továbbítja pótlólag.)

## <a name="analyzing-data-and-diagnosing-issues"></a>Az adatok elemzése és a problémák diagnosztizálása

A monitorozási és diagnosztikai folyamat fontos részét képezi a gyűjtött adatok elemzése a rendszer átfogó állapotát mutató kép létrehozása érdekében. Már meg kellett határoznia a KPI-ket és a teljesítmény mérőszámait, és fontos megértenie, hogy a gyűjtött adatokat hogyan strukturálhatja az elemzési igényeknek megfelelően. Fontos továbbá azt is megértenie, hogy a különböző mérőszámokban és naplófájlokban rögzített adatok korrelálása hogyan történik, mivel ezek az információk kulcsszerepet játszhatnak az események sorozatának nyomon követésében, és segíthetnek diagnosztizálni a felmerülő problémákat.

A [Rendszerállapot-adatok konszolidálása](#consolidating-instrumentation-data) szakaszban leírtak szerint a rendszer egyes részeire vonatkozó adatok rögzítése helyben történik, azonban általában egyesíteni kell azokat a rendszer részét képező egyéb helyeken keletkező adatokkal. Az ilyen információk esetén a korrelálást gondosan kell végezni, hogy az adatok egyesítése pontos legyen. Egy adott művelethez kapcsolódó használati adatok például magukban foglalhatnak egy olyan webhelyet futtató csomópontot, amelyhez a felhasználó csatlakozott, a művelet keretében elért önálló szolgáltatást futtató másik csomópontot, valamint egy harmadik csomóponton található adattárat. Ezeket az információkat össze kell kötni, hogy átfogó képet adjanak a művelet erőforrás- és feldolgozásikapacitás-használatáról. Az adatok bizonyos mértékű előzetes feldolgozása és szűrése már az adatokat rögzítő csomóponton megtörténhet, az összesítés és formázás azonban nagyobb valószínűséggel a központi csomóponton megy végbe.

### <a name="supporting-hot-warm-and-cold-analysis"></a>Forró, meleg és hideg elemzés támogatása

Az adatok megjelenítési, jelentéskészítési és riasztási célú elemzése és újraformázása összetett folyamat lehet, amely saját erőforrásokat használ. A monitorozás egyes formái esetében az időtényező kritikus fontosságú, és az adatok azonnali elemzését igénylik. Ez a *forró elemzés*. Példák erre a riasztásokhoz és a biztonsági monitorozás egyes szempontjaihoz (például a rendszer elleni támadás észlelése) szükséges elemzések. Az ilyen célokból szükséges adatoknak gyorsan elérhetőnek és strukturáltnak kell lenniük a hatékony feldolgozás érdekében. Bizonyos esetekben szükség lehet az elemzések feldolgozását az adatokat tároló csomópontokra áthelyezni.

Más elemzési formák esetében az időtényező kevésbé fontos, és bizonyos számításokra és összesítésre lehet szükség a nyers adatok fogadását követően. Ezt nevezzük *meleg elemzésnek*. A teljesítményelemzés gyakran ebbe a kategóriába esik. Ebben az esetben egyetlen, elszigetelt teljesítményesemény statisztikailag valószínűleg semmilyen jelentőséggel nem bír. (Okozhatta egy hirtelen megugrás vagy valamilyen hiba.) Az események sorozataiból származó adatok megbízhatóbb képet adnak a rendszer teljesítményéről.

A meleg elemzés segítséget nyújthat az állapotproblémák diagnosztizálásához is. Az állapotesemények feldolgozása általában forró elemzéssel történik, és az esemény azonnal létrehoznak riasztást. Az operátornak képesnek kell lennie elemezni az állapotesemény okait a meleg folyamatból származó adatok vizsgálatával. Ezeknek az adatoknak az állapoteseményt okozó problémához vezető eseményekkel kapcsolatos információkat kell tartalmazniuk.

A monitorozás bizonyos típusai hosszabb távú adatokat állítanak elő. Ez az elemzés egy későbbi időpontban is végrehajtható, akár egy előre meghatározott ütemezés szerint is. Bizonyos esetekben az elemzés során hosszabb időn keresztül rögzített, nagy mennyiségű adat összetett szűrését kell végrehajtani. Ezt nevezzük *hideg elemzésnek*. A legfontosabb követelmény, hogy az adatok biztonságos helyen legyenek tárolva a rögzítést követően. A használat monitorozásához és a naplózáshoz a rendszer állapotáról rendszeres időközönként készült pontos képekre van szükség, azonban az ilyen állapotadatoknak nem kell azonnal rendelkezésre állniuk feldolgozásra az adatgyűjtést követően.

Az operátor hideg elemzéssel is biztosíthatja az adatokat a prediktív állapotelemzéshez. Az operátor az aktuális állapotadatokat (ezek a forró elemzési folyamatból származnak) egy megadott időszakban gyűjtött előzményadatokkal kiegészítve derítheti fel azokat a trendeket, amelyek hamarosan állapotproblémákat okozhatnak. Ezekben az esetekben riasztás létrehozása lehet szükséges, hogy korrekciós műveleteket lehessen végrehajtani.

### <a name="correlating-data"></a>Az adatok korrelálása

A rendszerállapot-figyelés által rögzített adatok pillanatképet biztosíthatnak a rendszer állapotáról, az elemzés célja azonban az, hogy ezek az adatok hasznosíthatók legyenek a gyakorlatban. Példa:

- Mi okozta az intenzív, rendszerszintű I/O-terhelést egy adott időpontban?
- Nagyszámú adatbázis-művelet miatt volt?
- Kimutathatók a hatásai az adatbázis válaszidejében, a tranzakciók másodpercenkénti számában, valamint az alkalmazás válaszidejében ugyanezen a csomóponton?

Ha igen, a terhelést csökkentő javítóintézkedés lehet az adatok szétosztása több kiszolgálóra. Ezenkívül kivételek jöhetnek létre a rendszer valamelyik szintjén fellépő hiba eredményeként. Egy adott szinten felmerülő kivétel gyakran egy újabb hibát vált ki az eggyel magasabb szinten.

Ebből kifolyólag az egyes szintek különböző típusú monitorozási adatait tudnia kell korrelálni a rendszer és az azon futó alkalmazások állapotát mutató átfogó kép létrehozásához. Ezután ezen információk segítségével eldöntheti, hogy a rendszer megfelelően működik-e vagy sem, és meghatározhatja a szükséges intézkedéseket a rendszer minőségének a javítására.

Az [Információk az adatok korrelálásához](#information-for-correlating-data) című szakaszban leírtak szerint gondoskodjon arról, hogy a nyers rendszerállapot-adatok elegendő környezeti és tevékenységazonosító-információkat tartalmazzanak az események korrelálásához szükséges összesítések támogatásához. Az adatok tárolása emellett különböző formátumokban történhet, ezért szükség lehet az elemzésükre, így szabványos formátumba konvertálhatók elemzéshez.

### <a name="troubleshooting-and-diagnosing-issues"></a>Hibaelhárítás és a problémák diagnosztizálása

A diagnosztika elvégzéséhez meg kell tudni állapítani a hibák vagy a váratlan működés okait, például a kiváltó okok elemzésével. Az ehhez szükséges információk általában az alábbiak:

- Az eseménynaplókból és nyomkövetési adatokból származó részletes információk a rendszer egészére vagy egy adott alrendszerre vonatkozóan egy adott időszakból.
- A rendszerben vagy egy adott alrendszerben egy adott időszak során fellépő bármilyen adott szintű kivételek vagy hibák eredményeként keletkezett teljes hívásláncok.
- A rendszerben bárhol vagy egy adott alrendszerben egy adott időszak során meghiúsult folyamatok összeomlási memóriaképei.
- Az összes felhasználó vagy a kiválasztott felhasználók által egy adott időszakban végrehajtott műveleteket rögzítő tevékenységnaplók.

Az adatok hibakeresési célú elemzéséhez gyakran a rendszer-architektúra és a megoldást alkotó különféle összetevők részletes műszaki ismerete szükséges. Ennek következtében az adatok értelmezéséhez, a hibák okainak megállapításához és a megfelelő javítási stratégia ajánlásához gyakran jelentős manuális beavatkozásra van szükség. Ezért célszerű lehet ezekről az információkról csupán egy másolatot tárolni az eredeti formátumban, és azt egy szakértőnek hideg elemzésre átadni.

## <a name="visualizing-data-and-raising-alerts"></a>Adatok megjelenítése és riasztások létrehozása

Minden monitorozási rendszer fontos eleme az adatok olyan formában való megjelenítése, amelynek révén az operátor gyorsan észreveheti a trendeket vagy a problémákat. Emellett fontos az is, hogy az operátort gyorsan értesíteni lehessen, ha valami olyan jelentős esemény következik be, amely figyelmet igényelhet.

Az adatok megjelenítése különféle formákat ölthet, beleértve az irányítópultokon való megjelenítést, a riasztásokat és a jelentéseket.

### <a name="visualization-by-using-dashboards"></a>Megjelenítés irányítópultok használatával

Az adatok megjelenítésének leggyakoribb módja az irányítópultok használata, amelyek diagramokkal, grafikonokkal vagy egyéb ábrákkal jelenítik meg az információkat. Ezek az elemek paraméterekkel rendelkeznek, és az elemzőknek minden helyzethez ki kell tudniuk választani a fontos paramétereket (például az időszakot).

Az irányítópultok hierarchikusan rendezhetők. A legfelső szintű irányítópultok átfogó képet adhatnak a rendszer egyes funkcióiról, azonban lehetővé teszik az operátor számára az adatok részletes elemzését. A rendszer lemezeinek átfogó I/O-teljesítményét ábrázoló irányítópulton például az elemzőnek meg kell tudnia tekinteni az egyes lemezek I/O-teljesítményét is, így megállapíthatja, hogy a nem egyenletes forgalomért egy vagy több eszköz-e a felelős. Ideális esetben az irányítópulton kapcsolódó információk is megjelennek, például az I/O-forgalmat létrehozó kérések forrása (a felhasználó vagy a tevékenység). Ezeknek az információknak a használatával meghatározható, hogy a terhelést egyenletesebben kell-e elosztani az eszközök között (és hogy miként), valamint hogy a rendszer jobban teljesítene-e az eszközök számának növelése esetén.

Az irányítópultokon színkódolás vagy valamilyen más vizuális jelölésrendszer is alkalmazható a rendellenes vagy a várt tartományon kívül eső értékek jelzésére. Az előző példával élve:

- A hosszabb időn keresztül a maximális kapacitáshoz közeli I/O-sebességgel üzemelő lemezek (forró lemez) vörös színnel jelölhetők.
- Az időnként rövid ideig a maximális kapacitásnak megfelelő I/O-sebességgel üzemelő lemezek (meleg lemez) sárga színnel jelölhetők.
- A normál használatot mutató lemezek zöld színnel jelölhetők.

Ahhoz, hogy egy irányítópult-rendszer hatékonyan működjön, rendelkeznie kell a feldolgozható nyers adatokkal. Ha saját irányítópult-rendszert hoz létre, vagy egy másik vállalat által fejlesztett irányítópultot használ, tisztában kell lennie azzal, hogy milyen rendszerállapot-adatokat kell gyűjtenie, milyen részletességi szinten, és hogyan kell formázni azokat, hogy az irányítópult fel tudja használni őket.

Egy jó irányítópult nem csupán megjeleníti az információkat, hanem azt is lehetővé teszi, hogy az elemző ad hoc kérdéseket tegyen fel az információkkal kapcsolatban. Egyes rendszerek felügyeleti eszközöket is biztosítanak, amelyek használatával az operátor végrehajthatja ezeket a műveleteket, és áttekintheti a mögöttes adatokat. Az információk tárolásához használt adattártól függően az is előfordulhat, hogy az adatok közvetlenül is lekérdezhetők, vagy további elemzés és jelentéskészítés céljával olyan eszközökbe importálhatók, mint a Microsoft Excel.

> [!NOTE]
> Az irányítópultokhoz való hozzáférést érdemes az arra jogosult személyekre korlátozni, mivel az információk üzleti szempontból bizalmasak lehetnek. Érdemes továbbá az irányítópultok alapjául szolgáló adatokat védeni, hogy a felhasználók ne módosíthassák azokat.

### <a name="raising-alerts"></a>Riasztások létrehozása

A riasztás a monitorozási és rendszerállapot-adatok elemzésének, és jelentős esemény észlelése esetén értesítések létrehozásának a folyamata.

A riasztás hozzájárul a rendszer megfelelő állapotának, válaszkészségének és biztonságának a fenntartásához. Fontos része minden olyan rendszernek, amely teljesítmény-, rendelkezésreállási és adatvédelmi garanciákat vállal a felhasználók felé, akiknek az adatok alapján azonnal cselekedni kell. Előfordulhat, hogy az operátort értesíteni kell a riasztást kiváltó eseményről. A riasztási használható az olyan rendszerfunkciók indítására is, mint az automatikus skálázás.

A riasztási általában az alábbi rendszerállapot-adatoktól függ:

- **Biztonsági események**. Ha az eseménynaplók azt jelzik, hogy ismételt hitelesítési és/vagy jogosultsági hibák lépnek fel, elképzelhető, hogy a rendszer támadás alatt áll, és az operátort tájékoztatni kell.
- **Teljesítmény-mérőszámok**. A rendszernek gyorsan kell reagálnia, ha egy adott teljesítmény-mérőszám meghalad egy megadott küszöbértéket.
- **Rendelkezésre állási adatait**. Hiba észlelése esetén szüksége lehet egy vagy több alrendszer gyors újraindítására, vagy egy másodlagos erőforrásra való feladatátvételre. Amennyiben egyes alrendszerekben ismétlődő hibák tapasztalhatók, ez komolyabb problémákra utalhat.

Az operátorok számos különféle csatornán keresztül kaphatják meg a riasztásokat, például e-mailben, személyhívón vagy SMS-üzenetben. A riasztások tartalmazhatnak a helyzet súlyosságára utaló adatokat is. Számos riasztási rendszer támogatja az előfizetői csoportokat, így az azonos csoportban lévő operátorok ugyanazokat a riasztásokat kapják meg.

A riasztási rendszereknek testreszabhatónak kell lenniük, és ehhez a mögöttes rendszerállapot-adatok megfelelő értékei szolgálhatnak paraméterként. Ez a megközelítés lehetővé teszi, hogy az operátor az adatok szűrésével a fontos küszöbértékekre és értékkombinációkra összpontosíthasson. Vegye figyelembe, hogy egyes esetekben a nyers rendszerállapot-adatok biztosíthatók a riasztási rendszer számára. Más esetekben azonban célszerűbb összesített adatokat biztosítani. (Egy riasztás kiváltására például akkor kerülhet sor, amikor a processzor kihasználtsága az elmúlt 10 percben meghaladta a 90 százalékot.) A riasztási rendszernek biztosított információknak tartalmaznia kell a megfelelő összegzési és környezeti adatokat is. Az ilyen adatok segítségével csökkenthető a vakriasztások esélye.

### <a name="reporting"></a>Jelentéskészítés

A jelentéskészítés segítségével egy általános áttekintés készíthető a rendszerről. Az áttekintés tartalmazhat előzményadatokat is az aktuális információk mellett. A jelentéskészítésre vonatkozó követelmények két nagyobb kategóriába sorolhatók: működési jelentések és biztonsági jelentések.

A működési jelentések általában az alábbi szempontokat érintik:

- Összesíti a statisztikákat, amellyel egy adott időszak során a rendszer egészének vagy adott alrendszernek az erőforrás-használat megértéséhez.
- Egy adott időszak során a rendszer egészének vagy adott alrendszernek erőforrás-használati trendek azonosítása.
- A rendszerben vagy egy adott alrendszerben egy adott időszak során kivételek monitorozása.
- Az üzembe helyezett erőforrások szempontjából az alkalmazás hatékonyságának meghatározása, és annak a teljesítmény szükségtelen csökkentése nélkül az erőforrások mennyisége (és a kapcsolódó költségek) csökkenthető-e.

A biztonsági jelentések a rendszer felhasználók általi használatának nyomon követésével foglalkoznak. Az alábbiakra terjedhetnek ki:

- **A felhasználói műveletek naplózása**. Ehhez rögzíteni kell az egyes felhasználók által végrehajtott kéréseket, azok dátum- és időadataival együtt. Az adatokat megfelelően strukturálni kell, hogy a rendszergazda gyorsan rekonstruálhassa az egyes felhasználók által az adott időszakban végrehajtott műveletek sorát.
- **Erőforrás használja felhasználó követési**. Ehhez rögzíteni kell, hogy egy adott felhasználó egyes kérései hogyan és milyen hosszan férnek hozzá a rendszert alkotó különböző erőforrásokhoz. A rendszergazdának az adatok használatával képesnek kell lennie a használatról szóló, felhasználókon alapuló, adott időszakra vonatkozó jelentés létrehozására, például számlázási célból.

Sok esetben kötegelt folyamatok is létrehozhatnak jelentéseket egy meghatározott ütemezés szerint. (A késés általában nem jelent problémát.) A jelentéseket azonban ad hoc alapon is létre kell tudni hozni, ha szükséges. Ha például az adatait egy relációs adatbázisban, például az Azure SQL Database-ben tárolja, egy eszköz, például az SQL Server Reporting Services segítségével kinyerheti és formázhatja az adatokat, és megjelenítheti azokat különféle jelentésekben.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

- [Útmutató az automatikus skálázáshoz](../best-practices/auto-scaling.md): azt ismerteti, hogy hogyan csökkenthetők a felügyeleti terhek, ha kevésbé van szükség olyan operátorra, aki folyamatosan monitorozza a rendszer teljesítményét, és döntéseket hoz az erőforrások hozzáadásáról vagy eltávolításáról.
- [Állapot végponti Monitorozását végző minta](../patterns/health-endpoint-monitoring.md) ismerteti, hogyan lehet egy alkalmazásban, amelyhez hozzáférhetnek külső eszközök elérhetővé tett végpontokon keresztül rendszeres időközönként működés-ellenőrzéseket implementálhat.
- [Elsőbbségi üzenetsor mintája](../patterns/priority-queue.md) bemutatja, hogyan várólistára helyezett üzenetek, hogy a sürgős kérések fogadása és a dolgozhatók fel kevésbé sürgős üzenetek előtt.

## <a name="more-information"></a>További információ

- [Microsoft Azure Storage felügyelete, diagnosztizálása és hibaelhárítása](/azure/storage/storage-monitoring-diagnosing-troubleshooting)
- [Azure: Telemetria alapjai és hibaelhárítása](https://social.technet.microsoft.com/wiki/contents/articles/18146.windows-azure-telemetry-basics-and-troubleshooting.aspx)
- [Diagnosztika engedélyezése az Azure Cloud Services és a Virtual Machines szolgáltatásban](/azure/cloud-services/cloud-services-dotnet-diagnostics)
- [Azure Redis Cache](https://azure.microsoft.com/services/cache/), [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) és [HDInsight](https://azure.microsoft.com/services/hdinsight/)
- [How to use Service Bus Queues](/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues) (A Service Bus-üzenetsorok használata)
- [Az SQL Server Business Intelligence használata az Azure Virtual Machines szolgáltatásban](/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-bi)
- [Riasztási értesítések fogadása](/azure/monitoring-and-diagnostics/insights-receive-alert-notifications) és [szolgáltatások állapotának nyomon követése](/azure/monitoring-and-diagnostics/insights-service-health)
- [Application Insights](/azure/application-insights/app-insights-overview)
