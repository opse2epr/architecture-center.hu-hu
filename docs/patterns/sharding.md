---
title: Horizontális skálázási minta
titleSuffix: Cloud Design Patterns
description: Egy adattárat horizontális partíció- vagy szilánkkészletté oszthat fel.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 52c0579e4b08aa18456e0cc5a26742aab39a1a7e
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010324"
---
# <a name="sharding-pattern"></a>Horizontális skálázási minta

[!INCLUDE [header](../_includes/header.md)]

Egy adattárat horizontális partíció- vagy szilánkkészletté oszthat fel. Ezzel javíthatja a skálázhatóságot nagy adatmennyiségek tárolásakor és elérésekor.

## <a name="context-and-problem"></a>Kontextus és probléma

Az egyetlen kiszolgáló által futtatott adattárolókra az alábbi korlátozások vonatkozhatnak:

- **Tárolóhely**. A nagyméretű felhőalkalmazások adattára várhatóan hatalmas mennyiségű adatokat fog tartalmazni, és adatok mennyisége idővel jelentősen megnőhet. Egy kiszolgáló jellemzően korlátozott méretű lemezterületet biztosít, a meglévő lemezeket azonban kicserélheti nagyobbakra, vagy további lemezekkel bővítheti a gépet az adatmennyiség növekedésével. A rendszer viszont végül el fog érni egy korlátot, amikor már nem lehet könnyen növelni a kiszolgáló tárolókapacitását.

- **Számítási erőforrások**. A felhőalkalmazásoknak nagyszámú egyidejű felhasználót kell támogatniuk, akik mindegyike az adattárban lévő információkat lekérő lekérdezéseket futtat. Előfordulhat, hogy az adattárat futtató egyetlen kiszolgáló nem tudja biztosítani a terhelést támogató szükséges számítási teljesítményt, ami a felhasználók számára hosszú válaszidőt és gyakori hibákat eredményez, amikor az adatok tárolását és lekérését megkísérlő alkalmazások túllépik az időkorlátot. Megoldás lehet a memória bővítése vagy a processzorok nagyobb teljesítményűre cserélése, a rendszer azonban el fog érni egy korlátot, amikor a számítási erőforrások már nem növelhetők tovább.

- **Hálózati sávszélesség**. Az egyetlen kiszolgálón futó adattárak teljesítményét végső soron az a sebesség határozza meg, amellyel a kiszolgáló képes a kérések fogadására és a válaszok elküldésére. Előfordulhat, hogy a hálózati forgalom mennyisége meghaladja a kiszolgálóhoz való csatlakozáshoz használt hálózat kapacitását, ami sikertelen kéréseket eredményez.

- **Földrajzi hely**. Jogi, megfelelőségi vagy teljesítménnyel kapcsolatos okokból, vagy pedig az adatelérés késésének csökkentése érdekében szükséges lehet az adott felhasználók által létrehozott adatokat a felhasználó régiójában tárolni. Ha a felhasználók különböző országokban vagy régiókban vannak, előfordulhat, hogy nem lehet egyetlen adattárolóban tárolni az alkalmazás összes adatát.

A nagyobb lemezkapacitás, feldolgozási teljesítmény, több memória és hálózati kapcsolat hozzáadásával történő függőlegesen skálázás késleltetheti néhány ilyen korlátozás hatását, valószínűleg azonban csak ideiglenes megoldást jelent. A nagyszámú felhasználót és nagy mennyiségű adatot támogató kereskedelmi felhőalkalmazásoknak szinte végtelen skálázásra kell képesnek lenniük, így a függőleges skálázás nem feltétlenül a legjobb megoldás.

## <a name="solution"></a>Megoldás

Ossza fel az adattárat horizontális partíciókra vagy szegmensekre. Mindegyik szegmens ugyanazzal a sémával rendelkezik, azonban az adatok saját különálló részhalmazát tartalmazzák. A szegmensek adattárnak számítanak (sok különböző típusú entitás adatait tartalmazhatják), amelyek a tárolócsomópontként működő kiszolgálón futnak.

Ez a minta az alábbi előnyökkel jár:

- A rendszer horizontális skálázásához további tárolócsomópontokon futó további szegmenseket adhat hozzá.

- A rendszer azonnali használatra kész hardvert használhat az egyes tárolócsomópontokhoz szükséges speciális és költséges számítógépek helyett.

- A számítási feladatoknak a szegmensek közötti elosztásával csökkentheti a versengést és növelheti teljesítményt.

- A felhőben a szegmensek fizikailag közel helyezhetők el az adatokat elérő felhasználókhoz.

Amikor az adatárat szegmensekre osztja, döntse el, mely adatok kerüljenek az egyes szegmensekbe. A szegmensek általában olyan elemeket tartalmaznak, amelyek az adatok egy vagy több attribútuma által meghatározott tartományba esnek. Ezek az attribútumok alkotják a szegmenskulcsot (amelyet néha partíciókulcsnak is neveznek). A szegmenskulcsnak statikusnak kell lennie. Nem alapulhat olyan adatokon, amelyek változhatnak.

A horizontális skálázás fizikailag rendezi az adatokat. Ha egy alkalmazás adatokat tárol és kér le, a horizontális skálázási logika az alkalmazást a megfelelő szegmenshez irányítja. A horizontális skálázási logika megvalósítható az alkalmazás adatelérési kódjának részeként, vagy pedig az adattároló rendszer valósíthatja meg, ha transzparens módon támogatja a horizontális skálázást.

Az adatok fizikai helyének a horizontális skálázási logikában való meghatározásával magas szinten szabályozható, hogy az egyes szegmensek mely adatokat tartalmazzák. Emellett lehetővé teszi azt is, hogy az adatok az alkalmazás üzleti logikájának átdolgozása lehessen migrálhatók, ha a szegmensekben lévő adatokat később újra el kellene osztani (például akkor, ha a szegmensek kiegyensúlyozatlanná válnak). Ennek ára a további adatelérési terhelés, amelyre az adatok helyének meghatározásához van szükség az adatok lekérésekor.

Az optimális teljesítmény és skálázhatóság biztosítása érdekében olyan módon kell felosztani az adatokat, amely megfelel az alkalmazás által végrehajtott lekérdezések típusának. Sok esetben nem valószínű, hogy a horizontális skálázási séma pontosan megfelel minden lekérdezés követelményeinek. Egy több-bérlős rendszerben például lehet, hogy egy alkalmazásnak a bérlőazonosítóval kell lekérnie bérlőadatokat, de előfordulhat, hogy ezeket az adatokat más attribútum, például a bérlő neve vagy helye alapján kell megkeresnie. Az ilyen helyzetek kezelése érdekében a leggyakrabban végrehajtott lekérdezéseket támogató szegmenskulccsal rendelkező horizontális skálázási stratégiát valósítson meg.

Ha a lekérdezések rendszeresen kérnek le adatokat az attribútumértékek kombinációjával, valószínűleg meghatározhat egy összetett szegmenskulcsot az attribútumok összekapcsolásával. Másik megoldásként egy mintával, például az [indextábla](./index-table.md) használatával teheti lehetővé az adatok gyors keresését olyan attribútumok alapján, amelyekre a szegmenskulcs nem terjed ki.

## <a name="sharding-strategies"></a>Horizontális skálázási stratégiák

A szegmenskulcs kiválasztása és az adatok szegmensek közötti elosztási módjának meghatározása három gyakori stratégia használatával történik. Vegye figyelembe, hogy nem kell egy az egyhez típusú egyezésnek lennie a szegmensek és az azokat futtató kiszolgálók között – egyetlen kiszolgáló több szegmenst is futtathat. A stratégiák az alábbiak:

**A keresési stratégia**. Ebben a stratégiában a horizontális skálázási logika olyan összerendelést valósít meg, amely az adatkéréseket az adatokat tartalmazó szegmenshez irányítja a szegmenskulccsal. A több-bérlős alkalmazásokban előfordulhat, hogy egy bérlő összes adatát egy szegmens tárolja a bérlő azonosítóját használva szegmenskulcsként. Ugyanazon a szegmensen osztozhat több, azonban egy adott bérlő adatai nem oszlanak meg több szegmens között. Az ábra a bérlőadatok bérlőazonosítókon alapuló horizontális skálázását mutatja be.

   ![1. ábra – Bérlőadatok horizontális skálázása a bérlőazonosítók alapján](./_images/sharding-tenant.png)

A szegmenskulcs és a fizikai tároló közötti összerendelés alapulhat fizikai szegmenseken, mely esetben minden szegmenskulcs egy fizikai partíciónak felel meg. A szegmensek újraegyensúlyozásának egy rugalmasabb technikája a virtuális particionálás, mely során a szegmenskulcsok ugyanannyi virtuális szegmensre vannak leképezve, amelyek viszont így kevesebb fizikai partíciónak felelnek meg. Ezen megközelítés esetén az alkalmazás egy virtuális szegmensre hivatkozó szegmenskulccsal keresi meg az adatokat, és a rendszer transzparens módon képezi le a virtuális szegmenseket fizikai partíciókra. A virtuális szegmensek és fizikai partíciók közötti összerendelés anélkül változhat, hogy az alkalmazáskódot más szegmenskulcsok használatára kellene módosítani.

**A tartományalapú stratégia**. Ez a stratégia ugyanabban a szegmensben csoportosítja a kapcsolódó elemeket, és szegmenskulcs szerint rendezi azokat – a szegmenskulcsok sorrendben követik egymást. Ez olyan alkalmazások esetén hasznos, amelyek gyakran kérnek le elemeket tartománylekérdezésekkel (olyan lekérdezésekkel, amelyek adott tartományba eső szegmenskulcsok adatelemeit adják vissza). Ha például egy alkalmazásnak rendszeresen meg kell keresnie egy adott hónap összes megrendelését, akkor ezek az adatok gyorsabban lekérhetők, ha egy hónap összes megrendelésének a tárolása dátum és idő szerint rendezve történik ugyanabban a szegmensben. Ha az egyes megrendeléseket eltérő szegmensben tárolná, egyenként kellene lekérni azokat nagyszámú pontlekérdezés (egyetlen adatelemet visszaadó lekérdezés) végrehajtásával. A következő ábra az adatok egymást követő halmazának (tartományának) szegmensben való tárolását mutatja be.

   ![2. ábra – Adatok egymást követő halmazának (tartományának) tárolása szegmenseken](./_images/sharding-sequential-sets.png)

Ebben a példában a szegmenskulcs egy összetett kulcs, amelynek a legjelentősebb eleme a megrendelés hónapja, amelyet a megrendelés napja és ideje követ. A megrendelések adatainak rendezése értelemszerűen történik új megrendelések létrehozásakor és a szegmenshez való hozzáadásukkor. Néhány adattár támogatja a két részből álló szegmenskulcsokat, amelyek a szegmenset azonosító partíciókulcs-elemet és a szegmensben lévő elemet egyedi módon azonosító sorkulcsot tartalmaznak. A szegmens általában a sorkulcs sorrendjének megfelelően tárolja az adatokat. A tartománylekérdezésekkel lekérhető és az együtt csoportosítandó elemek olyan szegmenskulcsot használhatnak, amelynek partíciókulcsa a szegmenskulccsal egyező értékű, sorkulcsa azonban egyedi értékkel rendelkezik.

**A kivonatolási stratégia**. Ennek a stratégiának az a célja, hogy csökkentse a kritikus pontok esélyét (olyan szegmensek, amelyek aránytalanul nagy mennyiségű terhelést kapnak). A stratégia olyan módon osztja el az adatokat a szegmensek között, amely egyensúlyt teremt az egyes szegmensek mérete és a szegmensekre háruló átlagos terhelés között. A horizontális skálázási logika az elemet tároló szegmenst az adatok egy vagy több attribútumának kivonata alapján számítja ki. A kiválasztott kivonatolási függvénynek egyenletesen kell elosztania az adatokat a szegmensek között, lehetőleg valamilyen véletlenszerű elem bevezetésével a számításba. A következő ábra a bérlőadatoknak a bérlőazonosítók kivonata alapján történő horizontális skálázását mutatja be.

   ![3. ábra – Bérlőadatok horizontális skálázása a bérlőazonosítók kivonata alapján](./_images/sharding-data-hash.png)

A kivonatolási stratégia más horizontális skálázási stratégiákkal szembeni előnyének megértéséhez gondolja végig, hogy az új bérlőket egymás után regisztráló több-bérlős alkalmazás, hogyan rendelné a bérlőket az adattárban található szegmensekhez. A tartományalapú stratégia használatakor az 1–n bérlők adatainak mindegyikét az A szegmens tárolná, az n+1–m bérlők adatainak mindegyikét a B szegmens tárolná, és így tovább. Ha a legutóbb regisztrált bérlők a legaktívabbak is, a legtöbb adattevékenység kevés szegmensben történik, ami kritikus pontokat okozhat. Ezzel szemben a kivonatolási stratégia a bérlőazonosítójuk kivonata alapján rendeli a bérlőket a szegmensekhez. Ez azt jelenti, hogy az egymást követő bérlők valószínűleg különböző szegmensekhez lesznek rendelve, ami elosztja a szegmensek közötti terhelést. Az előző ábra ezt mutatja be az 55. és az 56. bérlőre vonatkozásában.

A három horizontális skálázási stratégia az alábbi előnyökkel és megfontolandó szempontokkal jár:

- **Keresési stratégia**. Nagyobb mértékű vezérlést nyújt a szegmensek konfigurálásának és használatának módja felett. A virtuális szegmensek használata csökkenti az adatok újraegyensúlyozásakor fellépő terhelést, mert új fizikai partíciók adhatók hozzá a számítási feladatok kiegyenlítése érdekében. A virtuális szegmens és a szegmenst megvalósító fizikai partíciók közötti összerendelés anélkül módosítható, hogy az hatással lenne az adatok tárolására és lekérésére szolgáló szegmenskulcsot használó alkalmazáskódra. A szegmenshelyek keresése további többletterhelést jelenthet.

- **Tartományalapú stratégia**. Könnyen megvalósítható, és jól működik a tartománylekérdezésekkel, mert azok gyakran több adatelemet kérhetnek le egyetlen szegmensből egyetlen művelet során. Ez a stratégia egyszerűbb adatkezelést kínál. Ha például egy adott régió felhasználói ugyanabban a szegmensben találhatók, a frissítések a helyi terhelés és az igényminta alapján ütemezhetők az egyes időzónákban. Ez a stratégia azonban nem biztosít optimális terheléselosztást a szegmensek között. A szegmensek újraegyensúlyozása nehéz feladat, és előfordulhat, hogy akkor sem oldja meg az egyenetlen terhelés problémáját, ha a tevékenységek többsége szomszédos szegmenskulcsokhoz kapcsolódik.

- **Kivonatolási stratégia**. Ez a stratégia nagyobb eséllyel teszi lehetővé az egyenletesebb adat- és terheléselosztást. A kérések továbbítása közvetlenül végezhető a kivonatolási függvény használatával. Nem kell leképezést fenntartani. Vegye figyelembe, hogy a kivonat kiszámítása többletterhelést okozhat. Emellett nehéz a szegmensek újraegyensúlyozása.

A leggyakoribb horizontális skálázási rendszerek a fent leírt megközelítések egyikét valósítják meg, figyelembe kell vennie azonban az alkalmazások üzleti követelményeit és az alkalmazások adathasználatának mintáit is. Egy több-bérlős alkalmazásban például:

- Az adatok horizontális skálázását végezheti számítási feladatok alapján. A kevéssé permanens bérlők adatait különálló szegmensekbe különítheti el. Ennek eredményeként növekedhet a többi bérlő adatelérésének sebessége.

- Az adatok horizontális skálázását végezheti a bérlők helye alapján. Egy adott földrajzi régióban található bérlők adatait biztonsági mentés és karbantartás céljából offline állapotba helyezheti az alacsony forgalmú időszakban, míg a többi régióban lévő bérlők adatai online és elérhetők maradnak a bérlők munkaidejében.

- Az értékes bérlők személyes, nagy teljesítményű, alacsony terhelésű szegmensekhez rendelhetők, ugyanakkor előfordulhat, hogy a kevésbé értékes bérlőknek kissé sűrűbb, forgalmas szegmenseken kell osztozniuk.

- A nagyfokú adatelkülönítést és adatvédelmet igénylő bérlők adatai teljesen különálló kiszolgálón tárolhatók.

## <a name="scaling-and-data-movement-operations"></a>Skálázási és adatáthelyezési műveletek

Minden horizontális skálázási stratégia a horizontális leskálázás, a horizontális felskálázás, az adatáthelyezés és az állapotfenntartás kezelésének különböző képességeivel és összetettségi szintjével jár.

A keresési stratégia a skálázási és adatáthelyezési műveletek online vagy offline, felhasználói szinten való végrehajtását teszi lehetővé. A módszer felfüggeszt néhány vagy minden felhasználói tevékenységet (esetleg a csúcsidőn kívüli időszakokban), az adatokat az új virtuális partícióra vagy fizikai szegmensre helyezi át, módosítja a leképezéseket, érvényteleníti vagy frissíti az ezen adatokat tároló gyorsítótárakat, majd engedélyezi a felhasználói tevékenység folytatását. Az ilyen típusú működés gyakran központilag kezelhető. A keresési stratégia megköveteli, hogy az állapot rendkívül jól gyorsítótárazható és replikabarát legyen.

A tartományalapú stratégia némileg korlátozza a skálázási és adatáthelyezési műveleteket, amelyeket általában akkor kell végrehajtani, amikor az adattár egy része vagy egésze offline állapotban van, mert az adatokat fel kell osztani és egyesíteni kell a szegmensek között. Előfordulhat, hogy a szegmensek újraegyensúlyozása céljából végrehajtott adatáthelyezés nem oldja meg az egyenetlen terhelés problémáját, ha a tevékenységek többsége szomszédos szegmenskulcsokhoz vagy az ugyanabban a tartományban lévő adatazonosítókhoz kapcsolódik. A tartományalapú stratégiához bizonyos állapot fenntartására is szükség lehet a tartományok fizikai partíciókra való leképezéséhez.

A kivonatolási stratégia összetettebbé teszi a skálázási és adatáthelyezési műveleteket, mivel a partíciókulcsok a szegmenskulcsok vagy adatazonosítók kivonatai. A kivonatolási függvényből vagy a megfelelő leképezések biztosítása érdekében módosított függvényből kell meghatározni az egyes szegmensek új helyét. A kivonatolási stratégiához azonban nincs szükség állapot fenntartására.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- A horizontális skálázás kiegészíti a particionálás többi formáját, például a függőleges particionálást és a funkcionális particionálást. Egy szegmens például tartalmazhat függőlegesen particionált entitásokat, egy funkcionális partíció pedig megvalósítható több szegmensként. További információ a particionálással kapcsolatban: [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx).

- Tartsa fenn az egyensúlyt a szegmensek között, hogy mindegyik hasonló mértékű I/O-forgalmat kezeljen. Adatok beszúrásakor és törlésekor rendszeresen újra kell egyensúlyozni a szegmenseket az egyenletes eloszlás biztosítása és a kritikus pontok esélyének csökkentése érdekében. Az újraegyensúlyozás drága művelet lehet. Az újraegyensúlyozás szükségességének csökkentése érdekében a növekedést szem előtt tartva tervezzen: győződjön meg arról, hogy mindegyik szegmens elegendő szabad hellyel rendelkezik-e a várt mennyiségű módosítás kezeléséhez. Stratégiákat és szkripteket is ki kell fejlesztenie a szegmensek gyors újraegyensúlyozására, ha az szükségessé válik.

- Stabil adatokat használjon a szegmenskulcshoz. Ha a szegmenskulcs megváltozik, előfordulhat, hogy a kulcsnak megfelelő adatelemnek mozognia kell a szegmensek között, így növekszik a frissítési műveletek által végzett munka mennyisége. Ezért ne alapozza a szegmenskulcsot olyan információkra, amelyek változhatnak. Ehelyett nem változó vagy jellegükből adódóan kulcsot formáló attribútumokat keressen.

- Gondoskodjon arról, hogy a szegmenskulcsok egyediek legyenek. Kerülje például az automatikusan növekvő mezők szegmenskulcsként való használatát. Néhány rendszerben az automatikusan növekvő mezők nem koordinálhatók a szegmensek között, ami miatt a különböző szegmenseken lévő elemek ugyanazzal a szegmenskulccsal rendelkezhetnek.

    >  Problémákat okozhatnak az egyéb olyan mezők automatikusan növekvő értékei is, amelyek nem szegmenskulcsok. Ha például automatikusan növekvő mezőkkel hoz létre egyedi azonosítókat, akkor két különböző szegmensben lévő két különböző elemhez ugyanaz az azonosító lehet hozzárendelve.

- Előfordulhat, hogy nem lehet olyan szegmenskulcsot tervezni, amely megfelel az adatokra irányuló minden lehetséges lekérdezés követelményeinek. Az adatok horizontális skálázásával támogassa a leggyakrabban végrehajtott lekérdezéseket, és szükség esetén hozzon létre másodlagos indextáblákat azon lekérdezések támogatásához, amelyek a szegmenskulcs részét nem képező attribútumokon alapuló feltételek használatával kérdezik le az adatokat. További információ: [indextábla minta](./index-table.md).

- A csak egy szegmenst elérő lekérdezések hatékonyabbak, mint az adatokat több szegmensből lekérő lekérdezések, ezért ne valósítson meg olyan horizontális skálázási rendszert, amely azt eredményezi, hogy az alkalmazások a különböző szegmensekben tárolt adatokat egyesítő nagyszámú lekérdezést hajtanak végre. Ne feledje, hogy egyetlen szegmens több entitástípus adatait is tartalmazhatja. Érdemes lehet denormalizálni az adatokat, hogy a gyakran együtt lekérdezett kapcsolódó entitások (például az ügyfelek és az általuk kezdeményezett megrendelések adatai) ugyanazon a szegmensben legyenek az alkalmazások által végrehajtott különálló olvasások számának a csökkentéséhez.

    >  Ha az egyik szegmensben lévő entitás egy másik szegmensben tárolt entitásra hivatkozik, az első entitás sémájának részeként adja meg a második entitás szegmenskulcsát. Ez hozzájárulhat a szegmenseken található kapcsolódó adatokra hivatkozó lekérdezések teljesítményének a javításához.

- Ha egy alkalmazásnak az adatokat több szegmensből lekérő lekérdezéseket kell végrehajtania, lehet, hogy az adatok lekérhetők párhuzamos tevékenységek használatával. Erre példa az elosztott lekérdezés, ahol a rendszer több szegmens adatait kéri le párhuzamosan, majd egyetlen eredményben összesíti azokat. Ez a megközelítés azonban elkerülhetetlenül összetetté teszi a megoldások adatelérési logikáját.

- Sok alkalmazás esetén hatékonyabb lehet több kisméretű szegmens létrehozása, mint kevés nagyméretű használata, mert ez több lehetőséget kínál a terheléselosztáshoz. Ez akkor is hasznos lehet, ha várhatóan az egyik fizikai helyről egy másikra kell szegmenseket migrálnia. A kis szegmensek gyorsabban áthelyezhetők, mint a nagyok.

- Gondoskodjon róla, hogy az egyes szegmenstároló csomópontok erőforrásai elegendőek legyenek a skálázhatósági követelmények kezelésére az adatméret és a feldolgozási sebesség tekintetében. További információért tekintse meg az [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx) „Partíciók tervezése skálázhatóságra” című szakaszát.

- Vegye fontolóra a referenciaadatok replikálását minden szegmensre. Ha egy szegmensből adatokat lekérő művelet statikus vagy lassan mozgó adatokra is hivatkozik ugyanazon lekérdezés részeként, adja hozzá ezeket az adatokat a szegmenshez. Az alkalmazás ezután könnyedén le tudja kérni a lekérdezés összes adatát anélkül, hogy további adatváltást kellene végrehajtania egy különálló adattárral.

    >  Ha a több szegmensben tárolt referenciaadatok megváltoznak, a rendszernek szinkronizálnia kell ezeket a változásokat minden szegmensben. A rendszer valamilyen fokú inkonzisztenciát tapasztalhat az ilyen szinkronizálás során. Ez esetben úgy kell megterveznie az alkalmazásokat, hogy képesek legyenek ennek kezelésére.

- Nehéznek bizonyulhat a hivatkozások integritásának és a szegmensek közötti konzisztenciának a fenntartása, ezért minimálisra kell csökkentenie a több szegmensben lévő adatokra hatással lévő műveletek számát. Ha egy alkalmazásnak több szegmensben található adatokat kell módosítania, mérje fel, hogy valóban szükség van-e az adatok teljes konzisztenciájára. Ehelyett egy, a felhőben gyakran alkalmazott megközelítés a végső konzisztencia megvalósítása. Az egyes partíciók adatainak frissítése külön történik, és az alkalmazáslogikának felelősséget kell vállalnia a frissítések mindegyikének sikeres befejezéséért, valamint egy végül konzisztensnek bizonyuló művelet futása során az adatok lekérdezéséből fakadó inkonzisztencia kezeléséért. A végül bekövetkező konzisztenciával kapcsolatos további információkat az [adatkonzisztenciát ismertető](https://msdn.microsoft.com/library/dn589800.aspx) szakaszban találja.

- Sok szegmens konfigurálása és kezelése kihívást jelenthet. Az olyan feladatokat, mint a monitorozás, biztonsági mentés, konzisztencia-ellenőrzés és naplózás, különálló, lehetőség szerint több helyen található szegmenseken és kiszolgálókon kell végezni. Ezek a feladatok valószínűleg megvalósíthatók szkriptekkel vagy más automatizálási megoldásokkal, azonban lehet, hogy ezek nem küszöbölik ki teljesen a további felügyeleti követelményeket.

- A szegmensek földrajzi helyhez köthetők, hogy az általuk tárolt adatok közel legyenek az adatokat használó alkalmazások példányaihoz. Ez a módszer jelentősen javíthatja a teljesítményt, azonban további szempontokat kell figyelembe venni olyan feladatok esetén, amelyeknek több, különböző helyeken található szegmenst kell elérniük.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Akkor használja ezt a mintát, ha egy adattárat valószínűleg egy adott tárolócsomópont számára elérhető erőforrásokon túl kell skálázni, vagy amikor javítani szeretné a teljesítményt egy adattárban tapasztalható versengés csökkentésével.

> [!NOTE]
A horizontális skálázás elsődleges célja a rendszerek teljesítményének és a skálázhatóságnak a javítása, mellékhatásként azonban a rendelkezésre állást is növelheti attól függően, hogy az adatok hogyan lettek külön partíciókra osztva. Egy partíción bekövetkező hiba nem akadályozza meg feltétlenül, hogy egy alkalmazás elérje a többi partíción tárolt adatokat, és egy operátor egy vagy több helyen végezhet karbantartást vagy helyreállítást anélkül, hogy egy alkalmazás összes adata elérhetetlenné váljon. További információ: [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx).

## <a name="example"></a>Példa

Az alábbi, C# nyelven írt példa SQL Server-adatbázisokat használ szegmensekként. Az egyes adatbázisok az alkalmazás által használt adatok egy részét tartalmazzák. Az alkalmazás a saját horizontális skálázási logikájával kéri le a szegmensek között elosztott adatokat (ez egy példa az elosztott lekérdezésre). Az egyes szegmensekben található adatok részleteit a `GetShards` nevű metódus adja vissza. Ez a metódus a `ShardInformation` objektumok enumerálható listáját adja vissza, amelyen a `ShardInformation` típus tartalmazza az egyes szegmensek azonosítóját és azon SQL Server kapcsolati sztringet, amellyel az alkalmazásnak a szegmenshez kell kapcsolódnia (a kapcsolati sztringek nem láthatók a kódpéldában).

```csharp
private IEnumerable<ShardInformation> GetShards()
{
  // This retrieves the connection information from a shard store
  // (commonly a root database).
  return new[]
  {
    new ShardInformation
    {
      Id = 1,
      ConnectionString = ...
    },
    new ShardInformation
    {
      Id = 2,
      ConnectionString = ...
    }
  };
}
```

Az alábbi kód bemutatja, hogyan használja az alkalmazás a `ShardInformation` objektumok listáját olyan lekérdezés végrehajtásához, amely mindegyik szegmensből párhuzamosan kéri le adatokat. A lekérdezés részletei nem láthatók, a példában a lekért adatok azonban egy sztringet tartalmaznak, amely információkat tartalmazhat, például egy ügyfél nevét, ha a szegmensek tartalmazzák az ügyfelek adatait. Az eredményeket egy `ConcurrentBag` gyűjteményben összesíti a rendszer az alkalmazás általi feldolgozáshoz.

```csharp
// Retrieve the shards as a ShardInformation[] instance.
var shards = GetShards();

var results = new ConcurrentBag<string>();

// Execute the query against each shard in the shard list.
// This list would typically be retrieved from configuration
// or from a root/master shard store.
Parallel.ForEach(shards, shard =>
{
  // NOTE: Transient fault handling isn't included,
  // but should be incorporated when used in a real world application.
  using (var con = new SqlConnection(shard.ConnectionString))
  {
    con.Open();
    var cmd = new SqlCommand("SELECT ... FROM ...", con);

    Trace.TraceInformation("Executing command against shard: {0}", shard.Id);

    var reader = cmd.ExecuteReader();
    // Read the results in to a thread-safe data structure.
    while (reader.Read())
    {
      results.Add(reader.GetString(0));
    }
  }
});

Trace.TraceInformation("Fanout query complete - Record Count: {0}",
                        results.Count);
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). A különböző szegmensek között elosztott adatok konzisztenciájának megőrzéséhez lehet szükséges. A cikk összefoglalja az elosztott adatok konzisztenciájának megőrzésével kapcsolatos problémákat, és bemutatja a különböző konzisztenciamodellek előnyeit és hátrányait.
- [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx). Az adattárak horizontális skálázása további problémákat vethet fel. Ez az útmutató az adattáraknak a skálázhatóság növelése, a versengés csökkentése és a teljesítmény optimalizálása érdekében a felhőben történő particionálásával kapcsolatos problémákat ismerteti.
- [Index Table minta](./index-table.md). A lekérdezések néha nem támogathatók teljes mértékben pusztán a szegmenskulcs kialakításának segítségével. Lehetővé teszi, hogy az alkalmazások gyorsan kérjenek le adatokat egy nagy adattárból a szegmenskulcstól eltérő kulcs megadásával.
- [Tényleges táblán alapuló nézet minta](./materialized-view.md). Bizonyos lekérdezési műveletek teljesítményének fenntartása érdekében célszerű materializált nézeteket létrehozni, amelyek egyesítik és összegzik az adatokat, különösen akkor, ha ezek az összegzett adatok a szegmensek között elosztott információkon alapulnak. Ismerteti, hogyan hozhatja létre és töltheti fel adatokkal ezeket a nézeteket.
