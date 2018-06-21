---
title: Gyorsítótárazási útmutató
description: A jobb teljesítmény és méretezhetőség gyorsítótárazás útmutatást.
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: fde1c3e8c65d357746e4ccaddebeebace943cf9d
ms.sourcegitcommit: 441185360db49cfb3cf39527b68f318d17d4cb3d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/19/2018
ms.locfileid: "27973144"
---
# <a name="caching"></a>Gyorsítótárazás

Gyorsítótárazás a közös technika, amely a célja, hogy a jobb teljesítmény és méretezhetőség, a rendszer. Ennek érdekében ideiglenesen másolása gyakran használt adatok gyors tárolók használata közeli található az alkalmazás. Ha az adatok gyors tároló közelebb található, mint az eredeti forrás az alkalmazáshoz, majd gyorsítótárazás jelentősen növelheti ügyfél alkalmazások válaszidejének előrejelzését adatok gyorsabban megtartásával.

Gyorsítótárazás esetén leghatékonyabb egy ügyfél példány ismételten olvassa be ugyanazokat az adatokat, különösen akkor, ha az eredeti adattárolóban alkalmazása a következő feltételeket:

* Viszonylag statikus marad.
* Lassú a gyorsítótár képest.
* Azt az versengés magas szintű.
* Az esetén távolságra hálózati késést okozhat, lassú a hozzáférést.

## <a name="caching-in-distributed-applications"></a>Az elosztott alkalmazások gyorsítótárazása
Elosztott alkalmazások általában megvalósítási valamelyike vagy mindegyike a következő stratégiák az adatokat:

* Használatával egy titkos gyorsítótárral, az adatok tárolási helye helyileg egy alkalmazás vagy szolgáltatás egy példányát futtató számítógépen.
* Megosztott gyorsítótárával, ami is elérhető, több folyamatok és/vagy gépek közös forrásként szolgáló használatával.

Mindkét esetben gyorsítótárazás hajtható végre az ügyféloldali és/vagy a kiszolgálóoldali. A folyamat, amely a felhasználói felületet biztosít a rendszerén, például egy webes böngésző vagy asztali alkalmazás ügyféloldali gyorsítótárazás történik.
A kiszolgálóoldali gyorsítótár végezhető el, hogy távolról futó üzleti szolgáltatásokat biztosít a folyamatot.

### <a name="private-caching"></a>Személyes gyorsítótárazása
A legalapvetőbb a gyorsítótár típus egy memórián belüli tároló. Egyetlen folyamat címterében használatban van, közvetlenül a kódot, a folyamat futó érhetők el. Ez a gyorsítótár típus rendkívül gyors eléréséhez. Mérsékelt mennyiségű statikus adatok tárolásához, mert a gyorsítótár méretének általában a folyamatot futtató gépen rendelkezésre álló memória mennyisége korlátozza egy rendkívül hatékony eszközöket is biztosítható.

Ha fizikailag lehet a memóriában, több információt gyorsítótárazzanak van szüksége, a helyi fájlrendszerben írhat a gyorsítótárazott adatokat. A lassabb, mint a memóriában tartott adatok eléréséhez, de továbbra is kell gyorsabb és megbízhatóbb, mint az adatok beolvasása a hálózaton keresztül.

Ha ez a modell egyidejűleg használó alkalmazás több példánya van, minden alkalmazáspéldány rendelkezik saját független gyorsítótár okozó saját az adatok másolatát.

Gondoljon egy gyorsítótár a múltban valamikor az eredeti adatok pillanatképként. Ha ezek az adatok nem statikus, valószínű, hogy a különböző alkalmazáspéldányok különböző verzióit a data tartsa a gyorsítótárak az. Ezért ezek a példányok által végzett ugyanabban a lekérdezésben adhat vissza eltérő eredményeket, az 1. ábrán látható módon.

![Egy memórián belüli gyorsítótárral, az alkalmazás különböző példányai használatával](./images/caching/Figure1.png)

*1. ábra: Az alkalmazás különböző példányát egy memórián belüli gyorsítótárral használatával*

### <a name="shared-caching"></a>Megosztott gyorsítótárazása
Megosztott gyorsítótárával használata segíthet enyhítse vonatkozik, hogy adatokat az egyes gyorsítótárát, amely akkor fordulhat elő, a memórián belüli gyorsítótárazáshoz eltérőek lehetnek. Megosztott gyorsítótárazás biztosítja, hogy különböző alkalmazáspéldányok tekintse meg a gyorsítótárazott adatokat nézetét. Ennek érdekében a gyorsítótár helyének egy külön helyen, általában egy külön szolgáltatás részeként futó, a 2. ábrán látható módon.

![Megosztott gyorsítótárával használatával](./images/caching/Figure2.png)

*2. ábra: Megosztott gyorsítótárával használatával*

Egy megosztott gyorsítótárazási megközelítés fontos előnye a méretezhetőséget biztosít. Számos megosztott gyorsítótár szolgáltatás kiszolgálófürtöt használatával lehet megvalósítani, és a szoftver, amely az adatok elosztja a fürt átlátható módon felhasználását. Az alkalmazáspéldány egyszerűen küld a cache service.
Az alkalmazás mögötti infrastruktúra felelős a gyorsítótárazott adatokat a fürt helyének meghatározása. A gyorsítótár további kiszolgálók hozzáadásával könnyedén méretezhető.

Két fő hátrányai a megosztott gyorsítótárazási megközelítés van:

* A gyorsítótár lassabb, mert azt már nem tartják helyileg alkalmazás-példányokhoz eléréséhez.
* A követelmény egy külön gyorsítótár szolgáltatás megvalósításához a megoldás lehet, hogy összetettebbé.

## <a name="considerations-for-using-caching"></a>Gyorsítótár használatának szempontjai
A következő szakaszok ismertetik részletesebben tervezéséről és a gyorsítótár használatával kapcsolatos szempontokat.

### <a name="decide-when-to-cache-data"></a>Döntse el, ha adatok gyorsítótárazása
Gyorsítótárazás jelentősen javíthatja a teljesítményt, a méretezhetőség és a rendelkezésre állási. A további adatok és a felhasználókat a nagyobb számú elérje ezeket az adatokat, annál nagyobb lesz a gyorsítótárazás előnyeit. Ennek oka az, gyorsítótárazás csökkenti a késést és nagy mennyiségű, az eredeti adattárolóban egyidejű kérelmek kezelése társított versengés.

Például egy adatbázis támogathatja az egyidejű kapcsolatok korlátozott számú. Adatok megosztott gyorsítótárával beolvasása, azonban ahelyett, hogy az alapul szolgáló adatbázis lehetővé teszi elérje ezeket az adatokat, akkor is, ha a rendelkezésre álló kapcsolatok száma jelenleg nagyon gyorsan kimerítették ügyfélalkalmazást. Továbbá ha az adatbázis nem érhető el, ügyfélalkalmazások lehet folytatja a gyorsítótárban tartott adatok felhasználásával.

Fontolja meg a ritkán (például az adatok, amelyek nagyobb arányban az olvasási műveletek, mint az írási műveletek) módosított, de gyakran beolvasott adatokat. Azonban nem ajánlott a mérvadó kritikus adatot tárol, a gyorsítótár használatát. Ehelyett győződjön meg arról, hogy a módosításokat, hogy az alkalmazás nem biztosít elveszítik mindig menti egy állandó adattárolóhoz. Ez azt jelenti, hogy ha a gyorsítótár nem érhető el, az alkalmazás továbbra is az adattár használatával, és nem vesznek el fontos információkat.

### <a name="determine-how-to-cache-data-effectively"></a>Határozza meg, hogyan hatékonyan gyorsítótárazza az adatokat
A kulcsot a használatával hatékonyan gyorsítótárat gyorsítótárhoz leginkább megfelelő adatok meghatározásakor, és a megfelelő időben, gyorsítótárazás értékű. Az adatok az igény szerinti először az alkalmazás lekéri azt a gyorsítótárba lehet hozzáadni. Ez azt jelenti, hogy az alkalmazás csak egyszer az adatok beolvasása a a tárolót, és, hogy később elérhetők a gyorsítótár használatával lehet teljesíteni.

Azt is megteheti a gyorsítótár részlegesen vagy teljesen feltölthetők adatok előre, általában az alkalmazás indításakor (összehangolása néven ismert megközelítés). Azonban nem feltétlenül ajánlott megvalósításához nagy gyorsítótár összehangolása, mert ez a megközelítés adhat az eredeti adattárolóban hirtelen és nagy terhelése, amikor az alkalmazás elindul.

Gyakran használati minták elemzésének segítségével határozza meg, hogy teljesen vagy részben feltöltse a gyorsítótárba, és a gyorsítótár kívánt adatok kiválasztásához. Például hasznos lehet a gyorsítótár a statikus felhasználói profil adatokkal rendezi az ügyfelek, akik használják az alkalmazást rendszeresen (lehet, hogy minden nap), de nem az ügyfelek, akik használják az alkalmazást csak hetente.

Gyorsítótárazás általában jól működik, amely nem módosítható, vagy ritkán végzett módosítások adatok. Például referencia jellegű információt, például a termék és díjszabási információkat e-kereskedelmi alkalmazás vagy a megosztott statikus erőforrásokat, amelyek költséges összeállításához. Néhány vagy az adatok tölthetők be a gyorsítótárba alkalmazások indításakor erőforrásainak terhelését a minimalizálása érdekében, és a teljesítmény javítása érdekében. Azt is megfelelő lehet egy olyan háttér folyamat, amely rendszeres időközönként frissíti az útmutató azt a gyorsítótárban lévő adatok folyamatosan naprakész adatokat tartalmazzon, vagy a gyorsítótárat frissíti, amely amikor referenciaadatok módosításokat.

Gyorsítótárazás érdemes kisebb dinamikus adatokat, habár van néhány kivétel ez szempont a (lásd a szakasz magas dinamikus adatok gyorsítótárazása a cikk későbbi részében olvashat). Ha az eredeti adatok rendszeresen vált, a gyorsítótárban lévő adatokkal elévülés nagyon gyorsan vagy növeli a gyorsítótár az eredeti adattárolóban való szinkronizálása a csökkenti a gyorsítótár hatékonyságát.

Ne feledje, hogy a gyorsítótár nem tartalmazza a teljes adatait, hogy egy entitás. Például ha adatelemet egy többértékű objektumot, például egy olyan nevét, címét és egyenleg banki ügyfél határozza meg, ezek közül néhány maradhat statikus (például a nevét és címét), mások (például az egyenleg) dinamikusabb lehetnek. Ezekben az esetekben hasznos lehet az adatok statikus részének gyorsítótárazza, és csak a fennmaradó adatok beolvasása (vagy kiszámításához) szükség.

Azt javasoljuk, hogy hajtsa végre a tesztelés és a használati Teljesítményelemzés előzetes feltöltési vagy igény szerinti betöltését, vagy mindkettőt, a gyorsítótár megfelelő meghatározásához. A döntéshez illékonyság és az adatok használati módja. Gyorsítótár-használat és Teljesítményelemzés különösen fontos az alkalmazásokat, amelyek nagy előforduló, és kiválóan méretezhető kell lennie. Például magas szinten méretezhető esetekben ésszerű rendezi a gyorsítótár csúcsidőben az adattár terhelésének csökkentése érdekében.

Gyorsítótárazás is segítségével kerülniük számítások az alkalmazás futása közben. Ha egy művelet átalakítja az adatokat, vagy összetett végez, akkor a művelet eredményét a gyorsítótár mentheti. Ha ugyanazt a számítást ezt követően szükség, az alkalmazás képes egyszerűen az eredmények visszakeresésére a gyorsítótárból.

Egy alkalmazás módosíthatja az adatokat, a gyorsítótár használatban van. Javasoljuk azonban a gyorsítótár adattárként átmeneti, tetszőleges időpontban tudta eltűnnek szokták érteni. Nem értékes adatok tárolása a gyorsítótár csak; Ügyeljen arra, hogy az információk, valamint az eredeti adattárolóban. Ez azt jelenti, hogy a gyorsítótár nem érhető el, ha minimálisra csökkenthető az adatvesztés esélyét.

### <a name="cache-highly-dynamic-data"></a>Magas dinamikus adatok gyorsítótárazása
Ha gyorsan változó információkat a állandó tárolóban vannak tárolva, azt a rendszer egy terhelésének adhat meg. Vegye figyelembe például olyan eszköz, amely folyamatosan a állapota vagy valamilyen egyéb mérési jelez. Ha egy alkalmazás nem gyorsítótárazza az adatokat az alapján, hogy a gyorsítótárban lévő adatokkal szinte mindig lesz elavult, az azonos szempont igaz, amikor tárolja, és ez az információ lekérése az adattár lehet. A mentéséhez, és ezek az adatok lehívása szükséges idő az módosulhatott.

Például ez esetben fontolja meg a dinamikus adatokat közvetlenül az állandó adattárban ahelyett, hogy a gyorsítótárban tárolja a előnyeit. Ha az adatok nem kritikus, és nem igényli a naplózás, majd nem számít, ha a alkalmi módosítás elvész.

### <a name="manage-data-expiration-in-a-cache"></a>A gyorsítótár adatok lejártának kezelése
A legtöbb esetben egy gyorsítótárban tartott adata, amely használatban van az eredeti adattárolóban adatok másolatát. Az eredeti adattárolóban lévő adatokat előfordulhat, hogy módosítása után az került a gyorsítótárba, így az a gyorsítótárazott adatok elavult. Számos gyorsítótárazási rendszer lehetővé teszi a gyorsítótár adatok lejárnak, és csökkenteni a időszakot, amelynek adatai pontatlanok lehetnek elavult konfigurálása.

Amikor gyorsítótárba helyezett adatok lejárnak, a rendszer eltávolítja a gyorsítótárban, és az alkalmazás kell lekérik az adatokat az eredeti adattárolóban (ezzel kockáztatja a újonnan lehívott adatokat vissza gyorsítótárba). Alapértelmezett lejárati házirendet állíthat be, amikor konfigurálja a gyorsítótárban. Sok gyorsítótár szolgáltatásokban akkor is is határozzák meg a lejárati időt, az egyes objektumok Ha tárolja őket programozott módon a gyorsítótárban.
Néhány gyorsítótárak lehetővé teszik a lejárati időt megadni, abszolút értéke, vagy egy mozgó érték, melynek következtében az elem a gyorsítótárból eltávolított, ha nem érhető el a megadott időn belül. Ez a beállítás felülbírálja a gyorsítótár kiterjedő elévülési szabályzatának, de csak a megadott objektum.

> [!NOTE]
> Fontolja meg a lejárati időt, a gyorsítótár és a benne található gondosan objektumok. Ha azt túl rövid, objektumok túl gyorsan lejár, és csökkenti a gyorsítótár használatának előnyeit. Ha az időszakban túl hosszú, azzal az adatok elévültek kockáztatja.
> 
> 

Lehetőség arra is, hogy a gyorsítótár előfordulhat, hogy megtelnek Ha adatokat hosszú ideig rezidens továbbra is engedélyezett. Ebben az esetben minden kérést új elemek hozzáadására a gyorsítótár okozhat egyes elemek kiürítés néven ismert folyamat kényszerített módon el kell távolítani. Gyorsítótár-szolgáltatások általában kizárása adatok (LRU) legkevésbé legutóbb használt időközönként, de általában ez a házirend felülbírálása és eltávolítandó elemek megakadályozása. Azonban ha elfogadni ezt a módszert használja, akkor kockáztatja meghaladja a gyorsítótár a rendelkezésre álló memória. Egy alkalmazás, amely megpróbálja vegyen fel egy elemet a gyorsítótár sikertelen lesz, és kivételt.

Egyes gyorsítótárazási megvalósítások előfordulhat, hogy olyan további kiürítés szabályzatokat. Többféle kiürítés házirendek. Ezek a következők:

* A legutóbb használt-szabályzat (az általános gyakorlat, hogy az adatokat nem szükséges ismét lesz).
* Egy első-first out házirend (legrégebbi adatokat először kizárt).
* Az explicit eltávolító házirendalapú (például a módosított adatok) kiváltott esemény.

### <a name="invalidate-data-in-a-client-side-cache"></a>Egy ügyféloldali gyorsítótárban lévő adatok érvénytelenné válnak
Használatban van egy ügyféloldali gyorsítótár adatokat általában tekinthető, amely adatokat szolgáltat az ügyfélnek a szolgáltatás felügyelete kívül. A szolgáltatás közvetlenül nem kényszerítheti ki egy ügyfél hozzáadásához vagy eltávolításához információk ügyféloldali gyorsítótárában.

Ez azt jelenti, hogy a rosszul konfigurált gyorsítótár továbbra is használja az elavult adatokat használó ügyfél. Például a gyorsítótár lejárati házirendje nem szabályszerű ügyfél elavult adatokat, amelyek a helyi gyorsítótárba helyezi az adatokat az eredeti adatforrás lapindexének változása után használhatja.

Ha a webes alkalmazás, amely adatokat szolgáltat a HTTP-kapcsolaton keresztül, implicit módon kényszerítheti a webes ügyfél (például egy böngésző vagy webproxyn) beolvasni a legfrissebb információkat. Ehhez az erőforrás URI megváltozása erőforrás frissítése. A webes ügyfelek általában az erőforrás URI az ügyféloldali gyorsítótár kulcsként használatához, ha megváltoztatja az URI Azonosítót, a webes ügyfél figyelmen kívül hagyja-e minden korábban erőforrás verziói a gyorsítótárban, és ehelyett beolvassa az új verziót.

## <a name="managing-concurrency-in-a-cache"></a>A gyorsítótár egyidejűségi kezelése
Gyorsítótárak gyakran tervezték, hogy egy alkalmazás több példánya is van osztva. Minden egyes alkalmazáspéldány olvashatják és módosíthatják az adatokat a gyorsítótárban. Az azonos egyidejűségi problémák merülnek fel, az összes megosztott tároló következésképpen is érvényesek a gyorsítótárba. Olyan helyzet, amikor egy alkalmazást kell módosítani a gyorsítótárban tartott adatait szükség lehet győződjön meg arról, hogy az alkalmazás egy példánya által végzett frissítések nem írja felül a másik példány által végzett módosításokat.

Attól függően, hogy az adatok természetét és ütközések valószínűségét is elfogadja párhuzamossági két módszer egyikét:

* **Optimistic.** Azonnal előtt frissíti az adatokat, az alkalmazás ellenőrzi a gyorsítótárban lévő adatok módosulásának lekérdezés óta. Ha az adatok továbbra is azonos, a módosítás hajtható végre. Ellenkező esetben az alkalmazásnak van határozza meg, hogy a frissítést. (Az üzleti logika, amely ehhez a döntéshez meghajtók lesz az alkalmazás-specifikus.) Ezt a módszert alkalmas a helyzetekben, ahol frissítések alkalomszerű, vagy ha ütközések nem valószínű, hogy mi történjen.
* **Pessimistic.** Amikor lekérdezi az adatokat, az alkalmazás zárolja a gyorsítótárban, megakadályozhatja, hogy egy másik példány a módosítás. Ez a folyamat biztosítja, hogy ütközések nem fordulhat elő, de blokkolására is képesek más esetekben kell feldolgozni ugyanazokat az adatokat. Pesszimista feldolgozási hatással lehet a megoldás a méretezhetőséget, és csak rövid élettartamú műveletek ajánlott. Ezt a módszert akkor lehet hasznos olyan esetekben, ahol ütközések valószínűbb, különösen akkor, ha egy alkalmazás több elem gyorsítótárában frissíti, és győződjön meg arról, hogy ezek a változások következetesen alkalmazzák.

### <a name="implement-high-availability-and-scalability-and-improve-performance"></a>Magas rendelkezésre állás és méretezhetőség megvalósítása, és a teljesítmény javítása
Kerülje a gyorsítótár elsődleges tárházaként adatok; Ez az a szerepe az eredeti adattárolóban, amelyből a rendszer a gyorsítótárból tölti fel. Az eredeti adattárolóban a megőrzése biztosításáért felelős.

Ügyeljen arra, hogy a megoldás megosztott gyorsítótár szolgáltatás rendelkezésre állását a kritikus függőségeket bevezetéséhez. Egy alkalmazás tudják is működjenek, ha a szolgáltatás, amely a megosztott gyorsítótár nem érhető el. Az alkalmazás nem kell lefagy vagy miközben a rendszer a gyorsítótár-szolgáltatást a sikertelen.

Ezért az alkalmazás ismeri fel a gyorsítótár-szolgáltatás rendelkezésre állását, és térhet vissza az eredeti adattárolóban, ha a gyorsítótár nem érhető el kell készíteni. A [megszakító mintát](http://msdn.microsoft.com/library/dn589784.aspx) akkor hasznos, ha ez a forgatókönyv kezelésére. A szolgáltatás, amely a gyorsítótár állíthatók helyre, és amint elérhetővé válik, a gyorsítótár tölteni, adatolvasás adattárolóból az eredeti, például a következő stratégiát, a [gyorsítótár-tartalékoljon mintát](http://msdn.microsoft.com/library/dn589799.aspx).

Azonban előfordulhat, a méretezhetőség hatása a rendszer, ha az alkalmazás visszaáll az eredeti adattárolóban átmenetileg nem érhető el a gyorsítótár esetén.
Az adattár helyreállítás alatt álló, amíg az eredeti adattárolóban sikerült kell swamped adatokat, és így a időtúllépések kérések, és nem sikerült a kapcsolat.

Vegye fontolóra egy helyi, saját gyorsítótár irányuló kérelem a megosztott gyorsítótár-et elérő összes alkalmazáspéldányok minden egyes példányában. Amikor az alkalmazás egy elemet kér le, azt is ellenőrzi először a saját gyorsítótárába, majd a megosztott a gyorsítótárba, és végül az eredeti adatok tárolásához. A helyi gyorsítótár adatok felhasználásával vagy a megosztott gyorsítótárban, vagy az adatbázis nem érhető el a megosztott gyorsítótárával esetén lehet megadni.

Ez a megközelítés megakadályozhatja, hogy a helyi gyorsítótárat a túl elévültek tekintetében a megosztott gyorsítótárával gondos konfigurációt igényel. Azonban a helyi gyorsítótár pufferként a Ha a megosztott gyorsítótár nem érhető el. 3. ábrán látható, ez a struktúra.

![Egy helyi, saját gyorsítótár használata megosztott gyorsítótárával](./images/caching/Caching3.png)
*3. ábra: a helyi, saját gyorsítótár használata megosztott gyorsítótárával*

Nagy méretű gyorsítótárak, amely viszonylag hosszú élettartamú adatok tárolására támogatása érdekében néhány gyorsítótár biztosítanak egy magas rendelkezésre állású lehetőség, amely megvalósítja az automatikus feladatátvételt, ha a gyorsítótár nem érhető el. Ez általában a megközelítés a gyorsítótárazott adatokat, hogy egy elsődleges gyorsítótár-kiszolgáló egy másodlagos gyorsítótár-kiszolgáló replikálása, és átvált a másodlagos kiszolgáló az elsődleges kiszolgáló meghibásodása esetén, vagy a kapcsolat elvész.

A több célhoz rendelt írása társított késés csökkentése érdekében a másodlagos kiszolgáló replikációs aszinkron módon során felmerülő adatot ír a gyorsítótár az elsődleges kiszolgálón. Ezt a megközelítést vezet a lehetősége, hogy néhány gyorsítótárazott adatok elvesztésével járhat meghibásodása van, de ezek az adatok arányának kis képest a gyorsítótár méretét.

Ha megosztott gyorsítótárával túl nagy, előnyös a gyorsítótárazott adatokat particionálásához esélyét csökkentheti a versengés, és a méretezhetőség javítása csomópontjai között lehet. Sok megosztott gyorsítótárak is támogatja a dinamikus hozzáadása (és eltávolítása) csomópontot, és az adatok egyensúlyba partíciók között. Ez a megközelítés előfordulhat, hogy tartalmaz, amely Fürtszolgáltatás, amely csomópontok gyűjteménye áll rendelkezésre ügyfélalkalmazások, zökkenőmentes, egyetlen gyorsítótár. Belsőleg azonban az adatok megvédheti a következő előre meghatározott telepítési stratégiát, amely egyenlően osztja szét csomópontok között. A [adatok particionálási útmutató](http://msdn.microsoft.com/library/dn589795.aspx) a Microsoft webhelyén particionálási stratégia lehet további információt nyújt.

Fürtszolgáltatás növelje a gyorsítótár rendelkezésre állását. Ha egy csomópont meghibásodik, a gyorsítótár fennmaradó továbbra is elérhetők maradnak.
Fürtszolgáltatás gyakran használt replikációs és feladatátvételi együtt. Minden csomópont replikálható, és a replika gyorsan online állapotba helyezhetők a csomópont meghibásodásakor.

Sok olvasási és írási műveletek valószínűleg egyetlen adatértékek vagy objektumokat. Azonban néha szükség lehet tárolja, vagy a nagy adatmennyiségek gyors beolvasása.
Például a gyorsítótár összehangolása magukban foglalhatják több száz vagy ezer cikkek írása a gyorsítótárba. Az alkalmazás számos kapcsolódó elemek beolvasása a gyorsítótárból a kérésben részeként is módosítania kell.

Sok nagy méretű gyorsítótárak kötegműveletek ezekből a célokból adja meg. Ez lehetővé teszi, hogy az egy kérelemhez elemek nagy mennyiségű becsomagolhatja ügyfélalkalmazást, és csökkenti a nagyszámú kis kérések végrehajtásához társított.

## <a name="caching-and-eventual-consistency"></a>Gyorsítótárazás és a végleges konzisztencia
A gyorsítótár-tartalékoljon minta működjön az alkalmazás, amely a gyorsítótárból tölti fel az példányát kell férnie a legtöbb legutóbbi és konzisztens legyen az adatok verziója. A rendszer, amely megvalósítja a végleges konzisztencia (például a replikált adatokat tároló) az nem lehet az eset.

Egy alkalmazás egy példánya nem sikerült módosítani egy adatelemet, és érvénytelenné válnak a gyorsítótárazott verzió elem. Előfordulhat, hogy az alkalmazás egy másik példánya megpróbálja olvassa el ezt az elemet a gyorsítótárból, aminek következtében a gyorsítótár-tévesztési, hogy azok az adatokat olvas a tárolót, és hozzáadja azt a gyorsítótárban. Ha az adattár nem lett teljesen szinkronban a más replikával, az alkalmazáspéldány sikerült olvasni, és a gyorsítótár, a régi értékű.

Az adatok konzisztenciájának kezelésére vonatkozó további információkért lásd: a [adatok konzisztencia ismertetése](http://msdn.microsoft.com/library/dn589800.aspx).

### <a name="protect-cached-data"></a>A gyorsítótárazott adatok védelme
A cache service függetlenül használja, fontolja meg, hogyan védi az adatokat a jogosulatlan hozzáféréstől gyorsítótárban tartott. Nincsenek két fő vonatkozik:

* A gyorsítótárban lévő adatok adatvédelmet.
* Az adatok védelme, mert a gyorsítótár és az alkalmazás által használt a gyorsítótár között zajló kommunikációról.

A gyorsítótárban lévő adatok védelme érdekében a cache service bevezetheti olyan hitelesítési mechanizmus, amely megköveteli, hogy alkalmazásokat adja meg az alábbiakat:

* Mely identitások férhetnek hozzá a gyorsítótárban lévő adatok.
* Milyen műveletek (olvasási és írási), amelyek az identitások elvégzéséhez jogosultak.

Amely terhelés csökkentése érdekében van társítva olvasott és írt adatok követően identitás írási vagy olvasási hozzáférés a gyorsítótárhoz, hogy identitás használhatja-e az adatokat a gyorsítótárban.

Korlátozza a hozzáférést a gyorsítótárazott adatok megfelelő részhalmazát kell, ha a következők valamelyikét teheti:

* A gyorsítótár felosztása partíciók (különböző gyorsítótár-kiszolgálók használatával), és csak hozzáférést identitások a partíciók, amelyek használatához engedélyezni kell.
* Titkosíthatja az adatokat, minden egyes részhalmazban különböző kulcsokkal, és adja meg a titkosítási kulcsok csak identitásokat tartalmaz, amelyek rendelkezzenek hozzáféréssel az egyes részhalmaza. Egy ügyfélalkalmazás továbbra is lehet beolvasni az összes adatot a gyorsítótárban, de csak lesz képes visszafejteni az adatokat, amelynek a kulcsokat tartalmaz.

Az adatok védelme kell azt tranzakciós átviteléhez is. Ehhez az szükséges, hogy a biztonsági szolgáltatásoktól függő a hálózati infrastruktúra, amely a gyorsítótárban való csatlakozáskor használandó ügyfélalkalmazások által biztosított. Ha a gyorsítótár megvalósítása a szervezeten belül helyszíni kiszolgáló használata, amely futtatja az ügyfélalkalmazások számára, majd a hálózat elkülönítését nem szükség lehet további lépéseket kell tennie. Ha a gyorsítótárban található távolról, és a TCP- vagy HTTP-kapcsolatot igényel (például az internethez) nyilvános hálózaton keresztül, vegye fontolóra SSL.

## <a name="considerations-for-implementing-caching-with-microsoft-azure"></a>Szempontok a gyorsítótárazást, ha a Microsoft Azure

[Azure Redis Cache](/azure/redis-cache/) nyílt forráskódú Redis gyorsítótár, amely egy Azure-adatközpontban szolgáltatásként fut megvalósítása. A gyorsítótárazási szolgáltatás, amely elérhető az összes Azure-alkalmazást, hogy az alkalmazás egy felhőalapú szolgáltatás, a webhely, vagy egy Azure virtuális gépen belüli biztosít. Gyorsítótárak megoszthatók ügyfélalkalmazások, amelyek rendelkeznek a megfelelő hozzáférési kulcsot.

Azure Redis Cache egy olyan nagy teljesítményű gyorsítótárazási megoldás, amely rendelkezésre állását, méretezhetőségét és biztonsági biztosít. Ez általában szolgáltatásként fut egy vagy több dedikált gépek elosztva. Megkísérli mértékű tárolása a memóriában gyors hozzáférés biztosítására használhatja. Ez az architektúra készült biztosít alacsony késéssel és magas teljesítmény ami csökkenti a lassú i/o-műveletek végrehajtásához.

 Azure Redis Cache összeegyeztethető számos olyan ügyfél-alkalmazások által használt különböző API-k. Ha meglévő alkalmazásokat, amelyek már az Azure Redis Cache helyileg futó, az Azure Redis Cache kapcsolatot biztosít gyors áttelepítés gyorsítótárazás a felhőben.


### <a name="features-of-redis"></a>A Redis jellemzői
 A redis több mint egy egyszerű gyorsítótár-kiszolgálót. Memóriában elosztott adatbázist biztosít olyan széles körű parancs-készletet, számos gyakori forgatókönyveket támogat. Ezek a jelen dokumentum szakasz használata a Redis gyorsítótárazás ismerteti. Ez a szakasz néhány fő funkciója Redis biztosító foglalja össze.

### <a name="redis-as-an-in-memory-database"></a>A memóriában adatbázisként redis
A redis egyaránt támogat olvasási és írási műveletet. A Redis írások védelme a rendszerhiba vagy rendszeresen tárolása egy helyi pillanatfelvétel fájlban vagy egy csak append naplófájlban. Ez nem a helyzet a sok gyorsítótárak (amelyet átmeneti adattároló).

 Összes írt aszinkron, és nem blokkolja az ügynökök adatok írásakor vagy olvasásakor. Ha Redis elkezd futni, az adatokat olvas a pillanatkép- vagy naplófájl fájlt, és használ a memóriabeli gyorsítótár összeállításához. További információkért lásd: [Redis-adatmegőrzés](http://redis.io/topics/persistence) a Redis-webhelyen.

> [!NOTE]
> A redis nem garantálja, hogy minden írási műveleteket a rendszer menti a katasztrofális hibája esetén, de a legrosszabb csak néhány másodperc érdemes adatok elveszhetnek. Ne feledje, hogy a gyorsítótár nem célja a mérvadó adatforrásként működni, és az alkalmazások, a gyorsítótár segítségével győződjön meg arról, hogy kritikus mentett adatok sikeresen egy megfelelő adattároló feladata. További információkért lásd: a [gyorsítótár-tartalékoljon mintát](http://msdn.microsoft.com/library/dn589799.aspx).
> 
> 

#### <a name="redis-data-types"></a>Redis-adattípusok
A redis egy kulcs-érték tároló, ahol értékek tartalmazhatnak egyszerű típusú vagy összetett adatszerkezetek, például a kivonatok, listák, és állítja be. Ezek az adattípusok atomi műveletek készlete támogatja. Kulcsok végleges vagy címkézett idő élettartamát, ekkor automatikusan eltávolított a gyorsítótárból a kulcs és a megfelelő értéket korlátozott lehet. Redis kulcsok és értékek kapcsolatos további információkért látogasson el a lap [Redis az adattípusokat és az absztrakt entitással egészült ki bemutatását](http://redis.io/topics/data-types-intro) a Redis-webhelyen.

#### <a name="redis-replication-and-clustering"></a>A redis replikációhoz és fürtözéshez
A redis támogatja a fő-és alárendelt replikációs rendelkezésre állásának és átviteli karbantartása. Az írási műveletek Redis fő csomópont egy vagy több alárendelt csomóponton előfordul replikálódnak. Olvasási műveletek a fő vagy a beosztottak bármelyikét szolgálhatók ki.

Esetén a hálózati partíció beosztottak továbbra is szolgál az adatok, és majd transzparens módon való újraszinkronizálás a master, ha a kapcsolat helyreállt. További részletekért látogasson el a [replikációs](http://redis.io/topics/replication) lap a Redis-webhelyen.

Redis is biztosít a fürtözés, amely lehetővé teszi a transzparens módon adatok particionálása a szilánkok kiszolgáló között, és a betöltés terjednek. Ez a funkció, javul a méretezhetőség, mert új Redis-kiszolgálók is hozzáadhatók, és az adatokat, mint a gyorsítótár méretének particionálni növeli.

Továbbá a fürt egyes kiszolgálóin fő-és alárendelt replikációval replikálható. Ez biztosítja a rendelkezésre állási a fürt minden csomópontja között. Fürtszolgáltatás és horizontális kapcsolatos további információkért látogasson el a [Redis-fürt oktatóprogram lap](http://redis.io/topics/cluster-tutorial) a Redis-webhelyen.

### <a name="redis-memory-use"></a>Redis memória használata
A Redis gyorsítótár korlátja véges, amely a gazdagépen rendelkezésre álló erőforrások függ. Egy Redis-kiszolgáló konfigurálásához, telepítheti a memória maximális mennyisége is megadhat. Egy Redis gyorsítótár lejárati időt, amely után automatikusan eltávolítja a gyorsítótárból is konfigurálhatja egy kulcsot. Ez a funkció is megakadályozzák a memóriabeli gyorsítótár való régi vagy elavult adatokat.

Memória kitölti-e, mint a Redis esetén automatikusan kizárása kulcsokkal és értékekkel házirendek száma következő. Az alapértelmezett érték LRU (legrégebben használt), de egyéb házirendek, például a kulcsok véletlenszerű kizárása vagy a kiürítési teljesen kikapcsolása (a elemek hozzáadására a gyorsítótár sikertelen lesz, ha betelt, mely eset kísérlet) is megadhat. A lap [használatával Redis, mint egy LRU gyorsítótár](http://redis.io/topics/lru-cache) nyújt részletesebb információt.

### <a name="redis-transactions-and-batches"></a>Redis tranzakciók és kötegek
A redis lehetővé teszi, hogy egy ügyfél-alkalmazás olvasása és írása az adatokat a gyorsítótárban, mint egy atomi tranzakciós műveletek sorozata elküldeni. A tranzakció a parancsok garantáltan egymás után futnak, és nincs más egyidejű ügyfelek által kiadott parancs fog interwoven, közöttük.

Azonban ezek nincsenek igaz tranzakciók, egy relációs adatbázisban hajtaná végre őket. Tranzakció-feldolgozást két szakaszból--áll az első akkor, ha a parancsok sorba, és a második pedig a parancsok futtatásakor. A parancsok a tranzakció alkotó parancs üzenetsor-kezelési szakaszában, az ügyfél által benyújtott. Ha valamilyen hiba akkor fordul elő, ezen a ponton (például szintaktikai hibát, vagy helytelen számú paraméterrel) majd Redis elutasítja a teljes tranzakció feldolgozása és figyelmen kívül hagyja azt.

A futtatási fázis során Redis sorrendben minden egyes sorban álló parancs végrehajtása. Ebben a fázisban a parancs sikertelen lesz, ha a Redis továbbra is fennáll, a következő sorba állított parancs, és nem állítja vissza már futó parancsok hatásait. A tranzakció egyszerűsített képernyőn segít a teljesítmény megőrzése, és a versengés által okozott problémák elkerülése érdekében.

A redis valósítja meg a konzisztencia fenntartása támogatására optimista zárolási űrlap. Tranzakciók és a redis gyorsítótárral zárolása kapcsolatos részletes információkért látogasson el a [tranzakciók lap](http://redis.io/topics/transactions) a Redis-webhelyen.

A redis is támogatja a kérelem nem tranzakciós kötegelés. A Redis-protokollt használó ügyfelek számára parancsainak elküldését a Redis-kiszolgáló lehetővé teszi az ügyfél küld a kérésben részeként műveleteket. Ez segít a hálózati csomag töredezettségének csökkentése érdekében. A köteg feldolgozása után minden parancs történik. Ha ezen parancsok bármelyikéhez az helytelen formátumú, akkor a program elutasítja (ami nem kerül sor a tranzakciók), de történik, a többi parancs. Még nincs garancia kapcsolatos a sorrendben, amelyben a kötegben parancsokat dolgoz fel.

### <a name="redis-security"></a>Biztonsági redis
A redis összpontosít tisztán adatok gyors hozzáférést biztosító, és célja, hogy csak megbízható ügyfelek által elérhető megbízható környezet futhat. A redis egy jelszó-hitelesítés alapján korlátozott biztonsági modellt támogatja. (Is lehet távolítsa el a hitelesítés teljesen, bár nem ajánlott ennek.)

Az összes hitelesített ügyfelek megosztása globális ugyanazt a jelszót, és elérheti az erőforrásokhoz. Ha átfogóbb bejelentkezési biztonságra van szüksége, meg kell valósítani a saját biztonsági réteg a Redis-kiszolgáló elé, és az összes ügyfélkéréseket kell érintenie a réteget. A redis nem számára nem megbízható és nem hitelesített ügyfeleket közvetlenül elérhetővé tehető.

Tiltsa le őket, vagy azokat átnevezése (és jogosultsággal rendelkező ügyfelek csak az új névvel rendelkező megadásával) korlátozzuk parancsok.

Redis közvetlenül nem támogatja bármely formájára vonatkozó adatok titkosítása, így minden kódolás ügyfélalkalmazások kell végrehajtani. Továbbá a Redis nem adja meg a transport security bármilyen. Ha kell azt a hálózaton keresztül zajlik védheti az adatokat, ajánlott végrehajtási egy SSL-proxy.

További tudnivalókért keresse fel a [biztonsági Redis](http://redis.io/topics/security) lap a Redis-webhelyen.

> [!NOTE]
> Azure Redis Cache nyújt a saját biztonsági réteg, amelyekkel az ügyfelek kapcsolódnak. Az alapul szolgáló Redis-kiszolgálók nem érhetők el a nyilvános hálózathoz.
> 
> 

### <a name="azure-redis-cache"></a>Azure Redis cache
Azure Redis Cache Redis-kiszolgálók az Azure adatközpontjában futó hozzáférést biztosít. A hozzáférés-vezérlés és adatvédelmet homlokzati funkcionál. A gyorsítótár az Azure portál használatával építhető ki.

A portál számos előre definiált konfigurációkat. A tartomány egy dedikált futtató 53 GB gyorsítótárában, hogy támogatja az SSL-kommunikáció (adatvédelmi) és a szolgáltatás fő-és alárendelt replikációt szolgáltatásiszint-szerződésben garantált 99,9 %-os rendelkezésre állás érdekében nélkül (nincs rendelkezésre állási garanciák) replikációs 250 MB méretű gyorsítótárat le a futó megosztott hardver.

Az Azure portál használatával, akkor is konfigurálja a kiürítés házirendet, a gyorsítótár, és a szerepköröket, meghatározott felhasználók hozzáadásával a gyorsítótárban való hozzáférést.  Ezeket a szerepköröket, amelyek meghatározzák a műveletek a tagok hajthat végre, a tulajdonos, közreműködő, és ahhoz való olvasóra tartalmazza. Például a tulajdonos szerepkör tagjai rendelkeznek a teljes felügyeletet gyakorolhat a gyorsítótárban (beleértve a biztonsági) és annak tartalmát, a közreműködői szerepkör tagjai és információt írhat a gyorsítótárban, és az Olvasó szerepkör tagjai is csak lekérdezni lehet adatokat a gyorsítótárból.

A legtöbb felügyeleti feladatok végrehajtására kerül sor, az Azure portálon keresztül. Emiatt a Redis szabványos verziójában elérhető felügyeleti parancsok közül sok nem érhető el, beleértve a programozott módon módosítsa a konfigurációt, állítsa le a Redis-kiszolgáló, további beosztottak konfigurálása vagy Kényszerített mentés adatok lemezre.

Az Azure-portál tartalmaz egy kényelmes grafikus megjelenítését, amely lehetővé teszi, hogy a gyorsítótár teljesítményének figyeléséhez. Például tekintheti meg és gyorsítótárbeli találatok száma és végeznek, a végrehajtás alatt álló kérelmek száma, az olvasási és írási, az adatmennyiség kapcsolatok száma. Ezen információk alapján is határozza meg a gyorsítótár hatékonyságát, és ha szükséges, váltani ettől eltérő konfigurációt, vagy módosítsa a kiürítés házirendet.

Emellett hozhat létre riasztásokat, amelyek egy e-mailek küldésére, ha egy vagy több kritikus metrikák egy várt tartományon kívül esnek. Például előfordulhat, hogy szeretne riasztást a rendszergazda Ha gyorsítótárbeli száma meghaladja a megadott érték az elmúlt órában, mert az azt jelenti, előfordulhat, hogy a gyorsítótár túl kicsi, vagy adatokat előfordulhat, hogy éppen dobható túl gyorsan.

A Processzor, memória és a gyorsítótár hálózathasználatáról is figyelheti.

További információt és példákat bemutatja, hogyan hozza létre és konfigurálja az Azure Redis Cache, látogasson el a [körül Azure Redis Cache Lap](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) meg az Azure blog.

## <a name="caching-session-state-and-html-output"></a>Gyorsítótárazás munkamenet-állapot és a HTML-kimenetében
Ha most felépítése ASP.NET webes alkalmazásokhoz, hogy az Azure webes szerepkörök használatával futtassa, mentheti munkamenet állapot információkat és az Azure Redis Cache HTML-kimenetet. Az Azure Redis Cache a munkamenetállapot-szolgáltatóját lehetővé teszi az ASP.NET webalkalmazások különböző példányai között munkamenet az információkat, és nagyon hasznos a webkiszolgáló farm olyan helyzetekben, ahol ügyfél-kiszolgáló kapcsolatot nem érhető el és gyorsítótárazási munkamenetadatok a memóriában nem lenne megfelelő.

Használata a munkamenetállapot-szolgáltatóját Azure Redis Cache kézbesíti számos előnnyel jár, beleértve:

* A nagy mennyiségű példánnyal ASP.NET webalkalmazás munkamenet-állapot megosztása.
* Továbbfejlesztett méretezhetőség biztosítása.
* Az azonos munkamenet-állapot adatainak ellenőrzése alatt, egyidejű hozzáférést támogató több olvasók és egyetlen írót.
* Tömörítés segítségével a memóriahasználat és a hálózati teljesítmény javításához.

További információkért lásd: [ASP.NET munkamenetállapot-szolgáltatóját az Azure Redis Cache](/azure/redis-cache/cache-aspnet-session-state-provider/).

> [!NOTE]
> Nem használatos a munkamenetállapot-szolgáltatóját Azure Redis Cache az ASP.NET-alkalmazások az Azure-alapú környezetben futtatható. A várakozási a gyorsítótárból, az Azure-on kívüli elérésének megszüntetheti által nyújtott az adatokat.
> 
> 

Hasonlóképpen a kimeneti gyorsítótár-szolgáltató az Azure Redis Cache lehetővé teszi a HTTP-válaszokat az ASP.NET webes alkalmazások által generált mentését. A kimeneti gyorsítótár-szolgáltató használata Azure Redis Cache javíthatja a leképezési összetett HTML-kimenetében alkalmazások válaszidejét. Alkalmazáspéldányok hasonló választ eredményező teheti a megosztott kimeneti töredék ez után a kimeneti HTML generálása helyett a gyorsítótár használatát. További információkért lásd: [az ASP.NET kimeneti gyorsítótár-szolgáltató Azure Redis Cache](/azure/redis-cache/cache-aspnet-output-cache-provider/).

## <a name="building-a-custom-redis-cache"></a>Egy egyéni Redis gyorsítótár létrehozása
Azure Redis Cache szolgáltatásba, illetve egy homlokzati az alapul szolgáló Redis-kiszolgálókra. Jelenleg konfigurációk készletét támogatja, de nem biztosít a Redis-fürtszolgáltatáshoz. Ha szüksége van, amely nem mutatja be az Azure Redis cache-(például a gyorsítótár 53 GB-nál nagyobb) speciális konfiguráció hozza létre, és a saját Redis állomáskiszolgáló Azure virtuális gépek használatával.

Ez a potenciálisan összetett folyamat, mert a több virtuális gépek alárendelt és csomópontok meghatalmazottjaként járhatnak el, ha azt szeretné, többszörözésére létrehozásához szükség lehet. Továbbá ha létre szeretne hozni egy fürtöt, akkor szüksége több főkiszolgálók és alárendelt kiszolgálók. A minimális fürtözött topológiát, amely magas szintű rendelkezésre állást és méretezhetőséget biztosít a három fő-és alárendelt kiszolgálók (fürt tartalmaznia kell legalább három fő csomópontok) értékpár formájában rendszerezett legalább hat virtuális gépek foglalja magában.

Minden fő-és alárendelt párt együtt zárja be a késés csökkentése érdekében érdemes kell elhelyezkedniük. Azonban minden álló párok csoportja futhat az különböző régiókban található különböző Azure adatközpontjaiban, ha közel az alkalmazásokat, amelyek valószínűleg használni a gyorsítótárazott adatok kereséséhez.  Például egy létrehozása és konfigurálása a Redis-csomópont egy Azure virtuális gépként fut, lásd: [a CentOS Linux virtuális gép az Azure-ban futó Redis](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx).

> [!NOTE]
> Vegye figyelembe, hogy a saját Redis gyorsítótár ily módon alkalmazza, ha Ön felelősséggel tartozik figyeléséhez, kezeléséhez és a szolgáltatás biztonságossá tétele.
> 

## <a name="partitioning-a-redis-cache"></a>A Redis gyorsítótár particionálás
A gyorsítótár particionálás magában foglalja a gyorsítótár felosztása több számítógép között. Ez a struktúra lehetővé teszi számos előnnyel egyetlen kiszolgálóval, beleértve:

* Olyan gyorsítótár létrehozását is nagyobb, mint egy kiszolgálón is tárolhatók.
* Osztja el az adatok között a kiszolgálók rendelkezésre állás javítása. Ha egy kiszolgáló meghibásodik vagy elérhetetlenné válik, az adatokat, amely tárolja nem érhető el, de a többi kiszolgálón továbbra is elérhető. A gyorsítótár ennek oka nem kritikus fontosságú a gyorsítótárazott adatokat csak átmeneti az adatok másolatát, amely az adatbázis használatban van. Egy olyan kiszolgálón, elérhetetlenné válik a gyorsítótárazott adatok helyette gyorsítótárazható egy másik kiszolgálón.
* A terhelés terjednek kiszolgáló, így javul a teljesítmény és méretezhetőség között.
* A felhasználók számára, amelyek férni, így csökkenti a késéseket lezárja Geolocating adatok.

A gyorsítótár a leggyakrabban használt particionálás formátuma horizontális. Ezt a stratégiát minden partíció (és horizontális skálázás) a Redis gyorsítótár önálló. Adatok horizontális logika, amely használatával megközelítés különböző terjesztheti az adatok segítségével egy adott partícióra van átirányítva. A [horizontális mintát](http://msdn.microsoft.com/library/dn589797.aspx) horizontális alkalmazásával kapcsolatos további információkat biztosít.

A Redis gyorsítótár particionálás alkalmazásához hajthatók végre a következő módszerek egyikét:

* *Kiszolgálóoldali lekérdezés útválasztást.* Ezzel a technikával az ügyfélalkalmazás egy kérést küld a Redis-kiszolgáló a gyorsítótárban (valószínűleg a legközelebbi kiszolgálót) alkotó. Minden egyes Redis-kiszolgáló tárolja, amely leírja, hogy azt tartalmazza, és információkat arról, hogy mely partíciók található más kiszolgálókon is tartalmaz a partíciós metaadatok. A Redis-kiszolgáló megvizsgálja az ügyfélkérés. Ha helyileg feloldható, hajtja végre a kért műveletet. Ellenkező esetben azt továbbítja a kérelmet be a megfelelő kiszolgálóra. Ez a modell fürtszolgáltatás Redis valósul meg, és további részletes leírását lásd a a [Redis-fürt oktatóanyag](http://redis.io/topics/cluster-tutorial) lap a Redis-webhelyen. A redis-fürtszolgáltatás ügyfélalkalmazások számára átlátható, és további Redis-kiszolgálók is hozzáadhatók a fürtöt (és az adatok újbóli particionálása) anélkül, hogy be újra az ügyfeleket.
* *Ügyféloldali particionálást.* Ebben a modellben az ügyfélalkalmazás logikája (valószínűleg a szalagtár formájában), amely kérelmek tartalmazza a megfelelő Redis-kiszolgálóhoz. Ez a megközelítés Azure Redis Cache használható. Hozzon létre több Azure Redis Cache-gyorsítótárak (egy mindegyik adatok partíció), és az ügyféloldali logika, amely a kéréseket a megfelelő gyorsítótárba végrehajtása. Ha a particionálási sémát (Ha további Azure Redis Cache-gyorsítótárak jönnek létre, például) változik, ügyfélalkalmazások módosítania kell újra kell konfigurálni.
* *Proxy támogatott particionálást.* A séma alkalmazások küldési kérelmek egy közbülső proxy szolgáltatás, amely tisztában van azzal, hogy az adatok particionálása, és ezután továbbítja a kérést a megfelelő ügyfél Redis kiszolgáló. Ez a módszer is használható az Azure Redis Cache; a proxy szolgáltatás Azure felhőalapú szolgáltatásként valósítható meg. Ez a megközelítés szükséges egy további szintű összetettség valósíthatja meg a szolgáltatást, és kérelmek hajtsa végre az ügyféloldali particionálást használnak, mint a hosszabb ideig is tarthat.

A lap [particionálására: hogyan adatok több Redis-példány között](http://redis.io/topics/partitioning) a Redis a webhely további információkat nyújt azokról a redis gyorsítótárral particionálás végrehajtására.

### <a name="implement-redis-cache-client-applications"></a>Redis gyorsítótár ügyfélalkalmazások megvalósítása
Számos programozási nyelven támogatása ügyfélalkalmazások redis. Ha a .NET-keretrendszer használatával hoz létre új alkalmazásokat, az ajánlott módszer a StackExchange.Redis ügyféloldali kódtár használata. Ebben a könyvtárban, amely a Redis-kiszolgálóhoz való kapcsolódás, parancsok küldése és fogadása válaszok részletes kivonatolja .NET-keretrendszer objektum modellt biztosít. Érhető el a Visual Studio NuGet-csomagként. Az Azure Redis Cache, vagy egy egyéni Redis gyorsítótár, a virtuális gép üzemeltetett való csatlakozáshoz használhatja ugyanazt a szalagtárat.

Egy Redis-kiszolgálóhoz való csatlakozáshoz használhat a statikus `Connect` metódusában a `ConnectionMultiplexer` osztály. Ezzel a módszerrel hoz létre a kapcsolat az ügyfélalkalmazás élettartama során használandó készült, és több egyidejű szálat használhatja ugyanazt a kapcsolatot. Kapcsolódjon újra, és ne válassza le minden alkalommal, amikor a Redis műveletet hajt végre, mert ez ronthatja a teljesítményt.

A kapcsolat paraméterek, például a címet a Redis-host és a jelszót is megadhat. Azure Redis Cache használatakor vagy az elsődleges vagy másodlagos kulcsot az Azure felügyeleti portál használatával az Azure Redis Cache létrehozott kell a jelszót.

Miután csatlakozott a Redis-kiszolgáló, az a Redis-adatbázisra, amelyet úgy működik, mint a gyorsítótár leírót ezt úgy szerezheti be. A Redis-kapcsolatot biztosít a `GetDatabase` ehhez metódust. Ezután elemek beolvasása a gyorsítótárból, és a gyorsítótár használatával adatok tárolása a `StringGet` és `StringSet` módszerek. Ezek a módszerek várt paramétereként egy kulcs, és térjen vissza az elem, vagy a gyorsítótár, amely rendelkezik a megfelelő érték (`StringGet`) vagy az elem felvétele a gyorsítótár a kulcshoz (`StringSet`).

Attól függően, hogy a Redis-kiszolgáló helyét sok művelet lehet, hogy fel Önnek némi késés, amíg a kiszolgáló átkerülnek a kérelmet, és választ küld vissza az ügyfélnek. A StackExchange kódtár biztosít aszinkron verzióinak nagy része a módszereket, amelyek továbbra is válaszol ügyfélalkalmazások segítségével érheti el. Ezek a módszerek támogatják a [feladatalapú aszinkron mintát](http://msdn.microsoft.com/library/hh873175.aspx) a .NET-keretrendszer.

A következő kódrészletet látható nevű metódus `RetrieveItem`. Azt mutatja be, a gyorsítótár-tartalékoljon minta alapján Redis és StackExchange szalagtár megvalósítása. A metódus karakterlánc-érték vesz igénybe, és megpróbálja beolvasni a megfelelő elem a Redis gyorsítótárt meghívásával a `StringGetAsync` metódus (aszinkron verzióját `StringGet`).

Ha az elem nem található, a mögöttes adatok forrás használatával beolvassa a `GetItemFromDataSourceAsync` metódus (amely egy helyi módszert, és nem része a StackExchange könyvtár). Ezután kerül a gyorsítótárba használatával a `StringSetAsync` , lekérhető gyorsabban legközelebb metódust.

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

A `StringGet` és `StringSet` metódusok nem korlátozódnak lekérése és karakterlánc-értékek tárolásához. A cikk, mint bájttömb szerializált tarthat. .NET-objektum mentése kell, ha szerializálni a byte adatfolyamként, és használja a `StringSet` metódus a gyorsítótárba írni.

Hasonló módon érheti el egy objektumot a gyorsítótárból használatával a `StringGet` metódus és a .NET objektumként deszerializálása azt. A következő kód bemutatja kiterjesztésmetódusok IDatabase interfész csoportja (a `GetDatabase` Redis kapcsolódási módszert adja vissza egy `IDatabase` objektum), és néhány példakód ezen módszerek írási és olvasási egy `BlogPost` objektum a gyorsítótárban:

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

Az alábbi kód bemutatja nevű metódus `RetrieveBlogPost` használó ezek kiterjesztésmetódusok írási és olvasási egy szerializálható `BlogPost` a gyorsítótár, a gyorsítótár-tartalékoljon mintát a következő objektumot:

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

Redis támogatja az parancs futószalagos, ha egy ügyfél alkalmazás több aszinkron kérést küld. A redis is vált a kérelmek ugyanazt a kapcsolatot használja helyett fogadására és válaszol a parancsok szigorú sorrendben.

Ez a megközelítés segít azáltal, hogy hatékonyabb használatát a hálózati késés csökkentésére. A következő kódrészletet mutat be, amely lekéri egyidejűleg két ügyfelek részleteit. A kód elküldi a két kérelmet, és néhány más feldolgozás (nincs ábrázolva) vár az eredmények előtt hajtja végre. A `Wait` a gyorsítótár-objektumának módszer hasonló a .NET-keretrendszer `Task.Wait` módszert:

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

Ügyfélalkalmazások, amelyek az Azure Redis Cache írásáról további információkért lásd: [Azure Redis Cache dokumentáció](https://azure.microsoft.com/documentation/services/cache/). További információk is érhető el: [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md).

A lap [folyamatok és multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) ugyanazt a webhelyet az aszinkron műveletek és a Redis és StackExchange szalagtár futószalagos további információt nyújt.  Ebben a cikkben a Redis gyorsítótár, a következő szakaszban néhány speciális módszert, amely használatban van a Redis gyorsítótár adatokkal alkalmazható példákat tartalmaz.

## <a name="using-redis-caching"></a>Redis gyorsítótár használata
A legegyszerűbb Redis aggályokat gyorsítótárazás használata kulcs-érték párok ahol az érték egy tetszőleges hosszúságú bináris adatokat tartalmazó nem értelmezett karakterlánc. (Lényegében bájttömb karakterláncként kezelhető szó). Ebben a forgatókönyvben lett mutatja be a szakasz megvalósítása Redis Cache ügyfélalkalmazások az ebben a cikkben.

Vegye figyelembe, hogy kulcsok nem értelmezett olyan adatokat is tartalmazhat, így minden bináris adatot kulcsként. Minél hosszabb a kulcsot meg kell azonban a több helyet fog tartani, amíg tárolja, és annál tovább fog tartani, amíg keresési műveletek végrehajtásához. A használhatóság és a könnyű karbantartási gondosan tervezése a kulcstérértesítések használatával, és használjon értelmezhető (de nem részletes) kulcsokat.

Például strukturált kulcsok, például a "felhasználói: 100" segítségével egyszerűen "100" helyett azonosító 100 az ügyfél a kulcsot jelöl. Ez a séma lehetővé teszi, hogy egyszerűen megkülönböztetni a tároló adattípusa különböző értékeket. A kulcs a azonosító 100 rendelés képviselő például a "rendelések: 100" kulcsot is használhat.

Egydimenziós bináris karakterláncok, leszámítva egy Redis kulcs-érték párokban szereplő érték is tárolható strukturáltabb információt, beleértve a listákat, és állítja be (rendezve és rendezetlen) csak. Redis biztosít átfogó parancsot, hogy ezek a típusok kezelhető, és ezek a parancsok számos StackExchange például egy ügyféloldali kódtára a .NET-keretrendszer alkalmazások. A lap [Redis az adattípusokat és az absztrakt entitással egészült ki bemutatását](http://redis.io/topics/data-types-intro) a Redis a webhely részletes áttekintést nyújt az ezek a típusok és a parancsok, amelyek segítségével kezelheti azokat.

Ez a szakasz néhány gyakori alkalmazási esetei ezeket az adattípusokat és parancsok foglalja össze.

### <a name="perform-atomic-and-batch-operations"></a>Hajtsa végre a atomi és kötegelt műveleteket
A redis atomi get és set műveletek karakterlánc-értékek több támogatja. Ezeket a műveleteket, távolítsa el a lehetséges versenyhelyzet veszélyek külön használatakor előforduló `GET` és `SET` parancsok. A rendelkezésre álló műveletek a következők:

* `INCR`, `INCRBY`, `DECR`, és `DECRBY`, amely műveleteket atomi növekmény és csökkentést egész numerikus értékek. A StackExchange kódtár biztosít túlterhelt verziói a `IDatabase.StringIncrementAsync` és `IDatabase.StringDecrementAsync` módszereket hajtsa végre ezeket a műveleteket, és térjen vissza az eredményül kapott érték, amely a gyorsítótárba. A következő kódrészletet mutatja be az alábbi módszerekkel:
  
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
* `GETSET`, amely lekéri az értéket, amelyet a kulcs van társítva, és nem módosítja azt egy új értékkel. A StackExchange könyvtár művelet keresztül elérhetővé teszi a `IDatabase.StringGetSetAsync` metódust. Az alábbi kódrészlet ezt a módszert példáját mutatja be. Ez a kód a kulcs "adatok: számláló" az előző példából tartozó aktuális értékét adja vissza. Majd azt visszaállítja a kulcs értéke nulla, az összes azonos művelet részeként:
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  string oldValue = await cache.StringGetSetAsync("data:counter", 0);
  ```
* `MGET`és `MSET`, amelyhez vissza vagy módosítsa egy karakterlánc-értékek beállítása egyetlen műveletben. A `IDatabase.StringGetAsync` és `IDatabase.StringSetAsync` módszerek túlterhelt támogatja ezt a funkciót, hogy a következő példában látható módon:
  
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

A Redis tranzakciók és kötegek szakaszban az ebben a cikkben leírtak Redis egyetlen tranzakció több műveletek is használhatja együttesen. A StackExchange könyvtár támogatást nyújt a tranzakciók keresztül a `ITransaction` felületet.

Létrehozhat egy `ITransaction` objektum használatával a `IDatabase.CreateTransaction` metódust. A tranzakció parancsok meghívásához által biztosított módszerekkel a `ITransaction` objektum.

A `ITransaction` felületet biztosít hozzáférést módszerek által elért hasonló a `IDatabase` felület, azzal a különbséggel, hogy a metódusok aszinkron jellegűek. Ez azt jelenti, hogy ezeket csak végre, ha a `ITransaction.Execute` meghívott metódus. A által visszaadott érték a `ITransaction.Execute` módszer azt jelzi, hogy a tranzakció sikeresen létrejött-e (igaz) vagy, ha (hamis) sikertelen volt.

A következő kódrészletet példáját mutatja be, hogy lépésekben, és csökkenti két számlálók ugyanabban a tranzakcióban részeként:

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

Ne feledje, hogy a Redis tranzakciók eltérően a relációs adatbázisok tranzakciók. A `Execute` metódus egyszerűen várólisták a tranzakció futtatható alkotó összes parancs, és ha valamelyiket rosszul megformázva majd a tranzakció le van állítva. Ha a parancsok sikeresen várólistára, minden parancs aszinkron módon futtatja.

Ha a parancs sikertelen, a többi feldolgozási továbbra is. Ha vissza kell igazolnia, hogy a parancs sikeresen befejeződött, kell-e a parancs beolvasása használatával a **eredmény** tulajdonság a megfelelő feladat, mint a fenti példában szereplő. Olvasás a **eredmény** tulajdonság letiltja a hívó szál, amíg a feladat befejeződött.

További információkért lásd: a [Redis-tranzakciók](https://stackexchange.github.io/StackExchange.Redis/Transactions) lap a StackExchange.Redis-webhelyen.

Kötegelt műveletek végrehajtásához, használhatja a `IBatch` StackExchange függvénytár felületet. Ez az interfész által elért hasonló módszerek készlete hozzáférést biztosít a `IDatabase` felület, azzal a különbséggel, hogy a metódusok aszinkron jellegűek.

Létrehozhat egy `IBatch` objektum használatával a `IDatabase.CreateBatch` metódust, és futtassa a kötegelt használatával a `IBatch.Execute` metódust, az alábbi példában látható módon. Ez a kód egyszerűen egy olyan karakterláncértéket, lépésekben és csökkenti az előző példában használt ugyanazokat a számlálókat beállítása, és megjeleníti az eredményeket:

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

Fontos megérteni, hogy eltérően a tranzakció kötegben a parancs futása sikertelen, mert az helytelen formátumú, ha a többi parancs előfordulhat, hogy továbbra is futtassa. A `IBatch.Execute` metódus nem ad vissza semmilyen arra utal, hogy a sikeres vagy sikertelen volt.

### <a name="perform-fire-and-forget-cache-operations"></a>Hajtsa végre a tűz és gyorsítótár-műveletekhez elfelejti
Támogatja a tűz redis, és műveletek elfelejti jelzők parancs használatával. Ebben a helyzetben az ügyfél egyszerűen indít el egy műveletet, de nem áll az eredmény érdekében, és nem várja meg a parancs végrehajtása. Az alábbi példa bemutatja, hogyan hajtsa végre a tűz INCR parancsot, és elfelejti műveletet:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
await cache.StringSetAsync("data:key1", 99);
...
cache.StringIncrement("data:key1", flags: CommandFlags.FireAndForget);
```

### <a name="specify-automatically-expiring-keys"></a>Adja meg az automatikusan lejáró kulcsok
Ha egy elemet a Redis gyorsítótár vannak tárolva, amely után az elem automatikusan törlődni fognak a gyorsítótárból időtúllépés is megadhat. Hogyan jóval több időt egy kulcs tartozik a lejárat előtt is lekérdezhet a `TTL` parancsot. Ez a parancs alkalmazásokhoz is elérhető legyen StackExchange használatával a `IDatabase.KeyTimeToLive` metódust.

A következő kódrészletet bemutatja, hogyan 20 másodperc lejárati idő egy kulcs, és a kulcs fennmaradó élettartama lekérdezése:

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

Is is beállíthatja a lejárati idő egy adott dátumot és időpontot a lejárati paranccsal, amelyik elérhető a StackExchange könyvtárban, mint a `KeyExpireAsync` módszert:

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
> Manuálisan eltávolíthat egy elemet a gyorsítótár a StackExchange a könyvtárból érhető DEL parancs használatával a `IDatabase.KeyDeleteAsync` metódust.

### <a name="use-tags-to-cross-correlate-cached-items"></a>Kereszt-korrelálja gyorsítótárazott elemek címkék használata
Egy Redis-készlet, amely egy kulcs megosztása több elem gyűjteménye. Egy készlet a SADD paranccsal hozhat létre. A csoportban lévő cikkek a SMEMBERS paranccsal kérheti le. A StackExchange függvénytár valósítja meg a SADD parancsot a `IDatabase.SetAddAsync` metódust, és a SMEMBERS parancs a `IDatabase.SetMembersAsync` metódust.

A SDIFF (set különbség), a SZINTERELŐ (set metszetének) és a SUNION (set union) parancsok segítségével új szalagoptimalizálási készleteket hozhat létre a meglévő készleteket kombinálhatja is. A szolgáltatás ezen műveletek a StackExchange könyvtárban a `IDatabase.SetCombineAsync` metódust. A metódusnak az első paraméter határozza meg a beállítási művelet végrehajtásához.

Az alábbi kódtöredékek bemutatják, hogyan beállítása gyorsan tárolja, és beolvassa a kapcsolódó elemek gyűjteményei hasznos lehet. Ezt a kódot használja a `BlogPost` megvalósítása Redis gyorsítótár ügyfélalkalmazások szakaszában az ebben a cikkben ismertetett típusa.

A `BlogPost` objektum négy mezőt tartalmaz – egy Azonosítót, cím, egy rangsorolási pontszám és címkék gyűjteménye. Az első kódrészletben látható a mintaadatok a C# listájának feltöltéséhez használt `BlogPost` objektumok:

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

A címkék az egyes tárolhatja `BlogPost` objektum a Redis gyorsítótár készletként, és rendelje hozzá minden Azonosítóját a `BlogPost`. Ez lehetővé teszi az alkalmazás gyorsan megtalálhatja a címkék, amelyek egy adott blogbejegyzést. Az ellenkező irányba keresés engedélyezéséhez, és minden blogbejegyzések, amelyek egy adott címke található, létrehozhat egy másik készlet, amely tárolja a blogbejegyzéseket, a kulcs a Címkeazonosító hivatkozik:

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

Ezen szerkezetek lehetővé teszik a nagyon hatékonyan számos gyakori lekérdezések végrehajtásához. Például található, és megjeleníti a blogbejegyzést 1 ilyen címkék:

```csharp
// Show the tags for blog post #1
foreach (var value in await cache.SetMembersAsync("blog:posts:1:tags"))
{
    Console.WriteLine(value);
}
```

Található, amelyek közösek a blogban található összes kódcímkének 1 és blog post 2 utáni végrehajtásával set metszetének, az alábbiak szerint:

```csharp
// Show the tags in common for blog posts #1 and #2
foreach (var value in await cache.SetCombineAsync(SetOperation.Intersect, new RedisKey[]
    { "blog:posts:1:tags", "blog:posts:2:tags" }))
{
    Console.WriteLine(value);
}
```

És egy adott címkét tartalmazó összes blogbejegyzések található:

```csharp
// Show the ids of the blog posts that have the tag "iot".
foreach (var value in await cache.SetMembersAsync("tag:iot:blog:posts"))
{
    Console.WriteLine(value);
}
```

### <a name="find-recently-accessed-items"></a>Keresés a nemrégiben elért elemek
A közös számos más alkalmazáshoz szükséges feladata a legtöbb nemrégiben elért elemek kereséséhez. Például egy bloggolás helyet érdemes a közelmúltban olvasási blogbejegyzések kapcsolatos információk megjelenítéséhez.

Ez a funkció a Redis lista használatával valósíthatja meg. Egy Redis lista több, azonos kulccsal rendelkező elem tartalmazza. A listában úgy működik, mint a kettős várólista. Elemek akkor leküldése vagy a lista végére, LPUSH (bal oldali leküldéses) és RPUSH (jobb oldali leküldéses) parancsok használatával. A lista másik végén elemek beolvasható a LPOP és RPOP parancsokkal. Elemek egy csoportjának a LRANGE és elrendezése paranccsal is vissza.

Az alábbi kódtöredékek bemutatják, hogyan végezheti el ezeket a műveleteket a StackExchange könyvtár használatával. Ezt a kódot használja a `BlogPost` az előző példákban típusa. Blogbejegyzés az Olvasás, felhasználó által a `IDatabase.ListLeftPushAsync` metódus leküldéses értesítések a kulcsot a Redis cache "blog:recent_posts" társított listáját, a blogbejegyzés címe.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:recent_posts";
BlogPost blogPost = ...; // Reference to the blog post that has just been read
await cache.ListLeftPushAsync(
    redisKey, blogPost.Title); // Push the blog post onto the list
```

További blogbejegyzések olvasható, mert a címben vannak fejlesztőre ugyanazt a listát. A lista a sorozatot, amely címének lettek hozzáadva van rendezve. A közelmúltban olvasási blogbejegyzések vannak a listán bal oldali vége felé. (Ha azonos blogbejegyzést csak egyszer olvasható, azt kell több bejegyzést a listában.)

A címek nemrég olvasási helyek használatával jelenítheti meg a `IDatabase.ListRange` metódust. Ez a módszer a kulcs a listában, egyfajta kiindulópontot és egy záró pontot tartalmazó vesz igénybe. A következő kód lekéri a 10 blogbejegyzések (elem 0-9) a bal szélső végén a lista címeit:

```csharp
// Show latest ten posts
foreach (string postTitle in await cache.ListRangeAsync(redisKey, 0, 9))
{
    Console.WriteLine(postTitle);
}
```

Vegye figyelembe, hogy a `ListRangeAsync` metódus nem elem eltávolítása a listából. Ehhez használhatja a `IDatabase.ListLeftPopAsync` és `IDatabase.ListRightPopAsync` módszerek.

Ha szeretné megakadályozni a lista növekvő határozatlan ideig, rendszeres időközönként a lista segítségével is selejtezett elemeket. Az alábbi kódrészlet bemutatja, hogyan távolítsa el az összes, de az öt bal szélső elemet a listából:

```csharp
await cache.ListTrimAsync(redisKey, 0, 5);
```

### <a name="implement-a-leader-board"></a>Alkalmazzon olyan vezető kártya
Alapértelmezés szerint a csoportban lévő elemek nem tartják megadott sorrendben. Egy rendezett sorozata az ZADD paranccsal hozhat létre (a `IDatabase.SortedSetAdd` metódus a StackExchange könyvtárban). Az elemek egy numerikus érték, egy pontszám, amely egy paramétert a parancshoz nevű használatával vannak rendezve.

A következő kódrészletet ad hozzá egy blogbejegyzést címe sorrendbe állított listáját. Ebben a példában minden egyes blogbejegyzést is, amely tartalmazza a prioritást, a blogbejegyzés pontszám mezőt tartalmaz.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:post_rankings";
BlogPost blogPost = ...; // Reference to a blog post that has just been rated
await cache.SortedSetAddAsync(redisKey, blogPost.Title, blogPost.Score);
```

A blog post címét és pontszámok növekvő pontszám sorrendben használatával kérheti le a `IDatabase.SortedSetRangeByRankWithScores` módszert:

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(redisKey))
{
    Console.WriteLine(post);
}
```

> [!NOTE]
> A StackExchange kódtár is biztosít a `IDatabase.SortedSetRangeByRankAsync` metódus, amely visszaadja az adatokat a pontszám sorrendben, de nem ad vissza a pontszámok.
> 
> 

Is beolvashatja csökkenő sorrendben a pontszámok elemeket, és a további paraméterek megadásával visszaadott tételszámának korlátozása a `IDatabase.SortedSetRangeByRankWithScoresAsync` metódust. A következő példában a címek és a felső 10 rangsorolt blogbejegyzések eredményét jeleníti meg:

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(
                               redisKey, 0, 9, Order.Descending))
{
    Console.WriteLine(post);
}
```

A következő példában a `IDatabase.SortedSetRangeByScoreWithScoresAsync` metódus, amely segítségével a rendszer visszairányítja az alábbiakhoz egy adott pontszám esik cikkek tartomány:

```csharp
// Blog posts with scores between 5000 and 100000
foreach (var post in await cache.SortedSetRangeByScoreWithScoresAsync(
                               redisKey, 5000, 100000))
{
    Console.WriteLine(post);
}
```

### <a name="message-by-using-channels"></a>Üzenet csatornák használatával
Egy adatgyorsítótár működött, leszámítva a Redis-kiszolgáló egy nagy teljesítményű publisher/előfizető mechanizmus üzenetküldést biztosít. Ügyfélalkalmazások kérhet le egy csatornát, és más alkalmazások vagy szolgáltatások üzenetek közzéteheti a csatornára. Alkalmazások előfizetés kapja meg ezeket az üzeneteket, és tud feldolgozni.

Redis biztosít az ügyfélalkalmazások fizet elő csatornák ELŐFIZETÉS parancsot. Ez a parancs egy vagy több csatornát, amikor az alkalmazás fogad üzeneteket neve vár. A StackExchange könyvtár magában foglalja a `ISubscription` felület, amely lehetővé teszi a .NET-keretrendszer alkalmazás előfizetés, és tegye közzé az csatornák.

Létrehozhat egy `ISubscription` objektum használatával a `GetSubscriber` metódus a Redis-kiszolgáló csatlakozik. Majd a használatával figyelni a csatornán üzenetek a `SubscribeAsync` metódus az objektum. Az alábbi példakód bemutatja, hogyan előfizetni a csatorna "üzenetek: blogPosts" nevű:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
await subscriber.SubscribeAsync("messages:blogPosts", (channel, message) => Console.WriteLine("Title is: {0}", message));
```

Az első paraméterben a `Subscribe` módszer a csatorna nevét. Ez a név a gyorsítótárban kulcsok által használt azonos szabályokat követi. A név tartalmazhat bináris adatokat, de ajánlott viszonylag rövid, jelentéssel bíró karakterláncok használatával biztosíthatja a megfelelő teljesítmény és karbantartási követelmények.

Azt is fontos megjegyezni, hogy a csatorna által használt névtér nem csatlakozik, kulcsok használatával. Ez azt jelenti, hogy lehet csatornák és kulcsok, azonos nevű, bár ez lehetséges, hogy az alkalmazás kódjában nehezebb karbantartása.

A második paraméter nem egy művelet delegált. Ez a delegált aszinkron módon fut, amikor egy új üzenet jelenik meg, a csatornán. Ez a példa egyszerűen megjeleníti az üzenet a konzol (az üzenet tartalmaz egy blogbejegyzést címe).

A csatorna való közzétételéhez egy alkalmazás paranccsal a Redis közzététele. A StackExchange kódtár biztosít a `IServer.PublishAsync` is végrehajthatja ezt a műveletet. A következő kódrészletet a "üzenetek: blogPosts" csatorna közzététele egy üzenetet jeleníti meg:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
BlogPost blogPost = ...;
subscriber.PublishAsync("messages:blogPosts", blogPost.Title);
```

Ismerje meg a közzétételi/feliratkozási mechanizmusa kapcsolatos számos olyan pontja van:

* Több előfizető előfizetni, ugyanazt a csatornát, és azok összes üzeneteket fogja kapni az adott csatornán közzétett.
* Előfizetők csak után azok előfizetett közzétett üzenetek fogadására. Csatornák nem pufferelt, és miután közzétette az üzenetet, a Redis-infrastruktúra leküldi mindegyik előfizető az üzenetet, és eltávolítja azt.
* Alapértelmezés szerint az üzenetek az elküldés vannak előfizetők által fogadott. Egy magas active rendszerben nagy számú üzenetek és sok előfizetők és a közzétevők üzenetek garantált szekvenciális kézbesítését lelassíthatja a rendszer teljesítményét. Ha minden üzenet független sorrendje nem lényeges, csak, párhuzamos feldolgozása engedélyezheti a Redis rendszer, így válaszkészsége javítása. Lehet ezt elérni StackExchange ügyfélprogram úgy, hogy a false értékre az előfizető által használt kapcsolat PreserveAsyncOrder:

```csharp
ConnectionMultiplexer redisHostConnection = ...;
redisHostConnection.PreserveAsyncOrder = false;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
```

### <a name="serialization-considerations"></a>Szerializálási kapcsolatos szempontok

Ha úgy dönt, a szerializálási formátum, fontolja meg a mellékhatásokkal között teljesítmény, együttműködés, versioning, a meglévő rendszerek, adatok tömörítése és Memóriaterhelést való kompatibilitást. Amikor teljesítmény értékelése, ne feledje, hogy referenciaalapok nagymértékben függ a környezetben. A munkaterhelések tényleges nem tükrözik, és újabb függvénytárak vagy a verzió nem tekinti. Nem minden forgatókönyvben egy "leggyorsabb" szerializálót van. 

Figyelembe kell venni néhány lehetőségek a következők:

- [Protokoll pufferek](https://github.com/google/protobuf) (is hívott protobuf) a strukturált adatok hatékonyan Google által fejlesztett szerializálási formátum. Adja meg az üzenet struktúrák szigorú típusmegadású definíciós fájlokat használja. A definíciós fájlok majd összeállítása a szerializálása és deszerializálása üzenetek nyelvspecifikus kódját. Protobuf használhatja meglévő RPC mechanizmusok keresztül, vagy hozhat létre a egy RPC szolgáltatás.

- [Apache Thrift](https://thrift.apache.org/) használ, egy hasonló módszert használja, szigorú típusmegadású definíciós fájlokat, és egy fordítási lépés a szerializálási kódot és az RPC szolgáltatásokat.  

- [Apache Avro](https://avro.apache.org/) protokoll Buffers és Thrift, hasonló szolgáltatásokat nyújt, de nincs összeállítása lépés. Ehelyett a szerializált adatok mindig tartalmazza a séma, amely leírja a struktúra. 

- [JSON](http://json.org/) emberek számára olvasható szövegmező használó nyílt szabvány. Széles körű többplatformos támogatást tartalmaz. JSON üzenet sémák nem használja. Folyamatban egy szöveges formátumú, nincs nagyon hatékony a hálózaton keresztül. Néhány esetben azonban, előfordulhat, hogy adnak vissza gyorsítótárazott elemek közvetlenül az ügyfél ebben az esetben tárolása JSON mentése volt egy másik formátumból deszerializálása során, és JSON majd szerializálása költsége HTTP-n keresztül.

- [BSON](http://bsonspec.org/) JSON hasonló struktúrával használó bináris szerializálási formátum. BSON úgy lett kialakítva, egyszerűsített, hogy könnyen és gyors szerializálása és deszerializálása, JSON viszonyítva kell lennie. Hasznos adat található hasonlóak mérete JSON. Attól függően, hogy az adatok egy BSON hasznos lehet kisebb vagy nagyobb, mint a JSON hasznos adatok között. BSON van néhány további adattípusok, amelyek nem érhetők el a JSON-ban, ilyen például a BinData (a byte tömbökben) és a dátum.

- [MessagePack](http://msgpack.org/) , amely a hálózaton keresztül kell számára az átvitelhez kompakt bináris szerializálási formátum. Nincsenek üzenet sémák vagy üzenet típusa ellenőrzése.

- [Kötés](https://microsoft.github.io/bond/) platformfüggetlen keretrendszere, amely sematizált adatok használata. Támogatja a többnyelvű szerializálása és deszerializálása. Figyelmet a jelentősebb eltérések a más rendszerekkel, az itt felsorolt támogatja az öröklési, típus és általánosítási. 

- [gRPC](http://www.grpc.io/) egy nyílt forráskódú Google által fejlesztett RPC rendszer. Alapértelmezés szerint protokoll pufferek az adatdefiníciós nyelv és az alapjául szolgáló üzenet interchange formátum használ.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintát is lehet a forgatókönyvhöz kapcsolódó, az alkalmazások gyorsítótárazás bevezetésekor:

* [Gyorsítótár-tartalékoljon mintát](http://msdn.microsoft.com/library/dn589799.aspx): Ebben a mintában az igény szerinti adatok betöltése a gyorsítótárba egy adattárból ismerteti. Ez a minta biztosítja az egységességet a gyorsítótárban tartott és az adatok az eredeti adattárolóban között is segíti.
* A [horizontális mintát](http://msdn.microsoft.com/library/dn589797.aspx) vízszintes particionálási tárolásakor méretezhetőség javítása érdekében, és nagy mennyiségű adat elérésekor végrehajtási információt nyújt.

## <a name="more-information"></a>További információ
* A [MemoryCache osztály](http://msdn.microsoft.com/library/system.runtime.caching.memorycache.aspx) oldalon, a Microsoft webhelyén
* A [Azure Redis Cache dokumentáció](https://azure.microsoft.com/documentation/services/cache/) oldalon, a Microsoft webhelyén
* A [Azure Redis Cache – gyakori kérdések](/azure/redis-cache/cache-faq) oldalon, a Microsoft webhelyén
* A [konfigurációs modell](http://msdn.microsoft.com/library/windowsazure/hh914149.aspx) oldalon, a Microsoft webhelyén
* A [feladatalapú aszinkron mintát](http://msdn.microsoft.com/library/hh873175.aspx) oldalon, a Microsoft webhelyén
* A [folyamatok és multiplexers](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) oldalon, a StackExchange.Redis GitHub-tárház
* A [Redis-adatmegőrzés](http://redis.io/topics/persistence) a Redis-webhelyen lap
* A [replikációs lap](http://redis.io/topics/replication) a Redis-webhelyen
* A [Redis-fürt oktatóanyag](http://redis.io/topics/cluster-tutorial) a Redis-webhelyen lap
* A [particionálására: hogyan adatok több Redis-példány között](http://redis.io/topics/partitioning) a Redis-webhelyen lap
* A [használatával Redis, mint egy LRU gyorsítótár](http://redis.io/topics/lru-cache) a Redis-webhelyen lap
* A [tranzakciók](http://redis.io/topics/transactions) a Redis-webhelyen lap
* A [biztonsági Redis](http://redis.io/topics/security) a Redis-webhelyen lap
* A [körül Azure Redis Cache Lap](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) oldalon, az Azure blog
* A [a CentOS Linux virtuális gép az Azure-ban futó Redis](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx) oldalon, a Microsoft webhelyén
* A [ASP.NET munkamenetállapot-szolgáltatóját az Azure Redis Cache](/azure/redis-cache/cache-aspnet-session-state-provider) oldalon, a Microsoft webhelyén
* A [az ASP.NET kimeneti gyorsítótár-szolgáltató Azure Redis Cache](/azure/redis-cache/cache-aspnet-output-cache-provider) oldalon, a Microsoft webhelyén
* A [Redis az adattípusokat és az absztrakt entitások bevezetés](http://redis.io/topics/data-types-intro) a Redis-webhelyen lap
* A [alapvető használati](https://stackexchange.github.io/StackExchange.Redis/Basics) a StackExchange.Redis-webhelyen lap
* A [Redis-tranzakciók](https://stackexchange.github.io/StackExchange.Redis/Transactions) oldalon, a StackExchange.Redis-tárház
* A [adatok particionálási útmutató](http://msdn.microsoft.com/library/dn589795.aspx) a Microsoft webhelyén

