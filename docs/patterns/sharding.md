---
title: "Horizontális"
description: "Adattároló felosztani vízszintes partíciók vagy szilánkok készlete."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 328483e24c75137f07576104d50dc59d426b8ac4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="sharding-pattern"></a>Horizontális minta

[!INCLUDE [header](../_includes/header.md)]

Adattároló felosztani vízszintes partíciók vagy szilánkok készlete. Ez javítja az méretezhetőséget, amikor tárolásához és eléréséhez nagy mennyiségű adatot.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egyetlen kiszolgálón található adattároló lehet a következő korlátozások vonatkoznak:

- **Tárolóhely**. A felügyeleti teendők központjaként felhőalapú alkalmazások tárolóban van kellett volna tartalmaznia túl nagy mennyiségű adatok, amelyek adott idő alatt jelentősen nő. Egy kiszolgáló általában csak a lemezes tárolás véges mennyiségű tartalmaz, de cserélje le a meglévő lemezek és nagyobb, vagy vegye fel a gépek további lemezek adatmennyiség növekedésével. Azonban a rendszer végül eléri a maximális ahol nem lehet könnyen növelje a tárolókapacitást a megadott kiszolgálón.

- **Számítási erőforrások**. A felhőalapú alkalmazások szükség, hogy számos egyidejű felhasználók, amelyek az adattárból adatbeolvasás lekérdezések futtatásához. Az adattár üzemeltető egyetlen kiszolgálót nem lehet a terhelés, ami hosszú válaszidejét a felhasználók és a gyakori hibák alkalmazásokat tárolásához és lekéréséhez adatok túllépi az időkorlátot, támogatásához szükséges számítási teljesítményt nyújt. Lehetséges, memória- vagy frissítési processzorok hozzáadása, de a rendszer eléri a határértéket során nem lehet növelni a számítási erőforrásokat minden további.

- **A hálózati sávszélesség**. Végső soron a tárolóban futó egyetlen kiszolgáló teljesítményének vonatkozik az átviteli sebesség a kiszolgáló fogadhatók kérések és válaszok küldhet. Akkor lehet, hogy a hálózati forgalom mennyiségének meghaladhatja a hálózat, a kiszolgáló, ami azt eredményezi, a sikertelen kérelmek való kapcsolódáshoz használt kapacitás.

- **A földrajzi**. Azoknak a felhasználóknak a jogi, a megfelelőségi vagy a teljesítmény szempontjából ugyanabban a régióban adott felhasználók által létrehozott adatok tárolására, vagy adatokhoz való hozzáférés késés csökkentése érdekében szükség lehet. Ha a felhasználók kerülnek át más országokból vagy régiókból között, hogy előfordulhat, hogy nem lehet az alkalmazás teljes adat egyetlen adattárolóban tárolja.

Skálázás függőleges adja hozzá a további lemezkapacitást, a feldolgozási teljesítmény, a memória és a hálózati kapcsolatok halaszthatják el a ezek a korlátozások némelyike hatásait, de valószínűleg csak egy ideiglenes megoldás. Egy kereskedelmi felhő alkalmazás képes nagyszámú felhasználó és a nagy mennyiségű adatot a méretezési szinte határozatlan ideig, képesnek kell lennie, így függőleges skálázás nem feltétlenül a legjobb megoldás.

## <a name="solution"></a>Megoldás

Az adattár felosztani vízszintes partíciók, sem a szilánkok. Minden egyes shard ugyanazon séma rendelkezik, de a saját az adatok különböző részét tartalmazza. A szilánkok a saját jobb (az adatok különböző típusú számos olyan entitás tartalmazhat), a tárolási csomópontnak működő kiszolgálón fusson a tárolóban.

Ebben a mintában használata a következő előnyökkel jár:

- További tárhely csomópontokon futó szilánkok hozzáadásával, méretezhető, a rendszer.

- A rendszer minden egyes tárhely csomópontot is használhat speciális és költséges számítógépek helyett a vásárolt hardver.

- Csökkentse a versengés, és a jobb teljesítmény érdekében a munkaterhelés terheléselosztás szilánkok között.

- A felhőben szilánkok lesz fér hozzá az adatokhoz, a felhasználók fizikailag közel található lehet.

A szilánkok felosztásával az adatok tárolja, amikor döntse el, mely adatokra minden shard kell helyezni. A szilánkok általában olyan elemeket tartalmaz, az adatok egy vagy több attribútum határozza meg a meghatározott tartományba tartoznak. Ezek az attribútumok alkotják a shard kulcsot (más néven a partíciós kulcs). A szilánkok kulcs lehet statikus. Azt nem kell adatok alapján, amely módosíthatja.

Horizontális fizikailag rendezi az adatokat. Ha egy alkalmazás tárolja, és olvassa ki az adatokat, a horizontális logika irányítja az alkalmazást a megfelelő szilánkcímtárban. A horizontális logika valósítható meg az adatok hozzáférési kódot az alkalmazás részeként, vagy azt lehetett végrehajtani az adatok tárolási rendszer Ha transzparens módon támogatja a horizontális.

A horizontális logika adatainak fizikai helyét absztrakt biztosít egy magas szintű szabályozást, amelyben a szilánkok milyen adatokat tartalmaznak. Azt is lehetővé teszi az adatok anélkül, hogy az üzleti logika egy alkalmazás reworking, ha az adatokat a szilánkok kell (például, ha a szilánkok válnak egyenetlen) később terjeszthető szilánkok közötti áttelepítése. Az kompromisszumot meghatározni, hogy a hely minden egyes adatelem, lekéri azt terhet szükséges további adatok hozzáférést.

Optimális teljesítményének és méretezhetőségének biztosításához fontos az adatok úgy, hogy megfelelő-e a lekérdezések, az alkalmazás végző típusú. Sok esetben nem valószínű, hogy a horizontális rendszer pontosan megegyeznek minden egyes lekérdezés követelményeinek. Például egy több-bérlős rendszerben az alkalmazás módosítania kell a bérlő azonosítójával bérlői adatok beolvasásához, de azt is módosítania kell az adatokat, például a bérlő nevét vagy helyét néhány más attribútum alapján kereshet. Ezek közül valamelyik helyzet kezelésére, egy horizontális skálázási stratégia végrehajtása, amely támogatja a leggyakrabban végrehajtott lekérdezések shard kulccsal.

Ha lekérdezések rendszeresen lekéréséhez használata az attribútumértékek adatok attribútumok összekapcsoló által valószínűleg határozhat meg egy összetett shard kulcsot. Másik megoldásként használhatja például a minta [Index táblázat](index-table.md) arra, hogy gyors értékét a adatokat olyan attribútumok, amelyek nem fedi le a shard kulcs alapján.

## <a name="sharding-strategies"></a>Horizontális stratégiák

Három stratégiák gyakran használják a shard kiválasztásakor kulcs és az adatok szét szilánkok módjának. Vegye figyelembe, hogy nem kell lennie egy-az-egyhez levelezés szilánkok és az azokat üzemeltető kiszolgálók között&mdash;egyetlen kiszolgáló, rendelkezhet több szegmensben osztják. A stratégia a következő:

**A keresési stratégia**. Ezt a stratégiát a horizontális logika valósítja meg, hogy a shard shard kulcs használatával adatokat tartalmazó továbbítja az adatokat kér. Egy több-bérlős alkalmazásban egy bérlő számára az adatok tárolódhat együtt egy shard shard kulcsként bérlői azonosító használatával. Több bérlő megosztott ugyanazt a shard, de egyetlen bérlő számára az adatok nem lehet több szilánkok elosztva. Az ábrán láthatja a horizontális bérlői adatokat a bérlői azonosító alapján.

   ![1. ábra – horizontális bérlői adatokat a bérlői azonosító alapján](./_images/sharding-tenant.png)


   A szilánkok kulcs és a tárolóhely közötti leképezést is alapulhat fizikai szilánkok, ahol minden shard kulcs van leképezve a fizikai partícióján. Azt is megteheti a szilánkok újraelosztás rugalmasabb módszer virtuális particionálás, ahol shard kulcsok virtuális szilánkok, amely pedig kevesebb fizikai partíciók leképezése azonos számú hozzárendelését. Ezt a módszert használja egy alkalmazás megkeresi, hogy egy virtuális shard hivatkozik a shard kulccsal adatok, és a rendszer transzparens módon van leképezve virtuális szilánkok fizikai partíciók. Egy virtuális shard és egy fizikai partíció közötti leképezést anélkül, hogy az alkalmazáskódban shard kulcsok különböző szabálykészleteket használnak módosítani módosíthatja.

**A tartomány stratégia**. Ezt a stratégiát kapcsolódó elemeket együtt ugyanazt a shard csoportosítja, és rendelések azokat a shard kulcs által&mdash;a shard kulcsokban szekvenciális. Érdemes gyakran beolvasása (lekérdezések tér vissza, amely a megadott tartományba esik shard kulcsok adatelemek) tartománya lekérdezések alkalmazásával készleteinek alkalmazások esetében. Például ha egy alkalmazás rendszeresen meg kell keresnie egy adott hónap összes megrendelések, ezek az adatok gyorsabban esetén lekérhetők ugyanazt a shard dátum és idő sorrendben összes rendelés egy hónapig tárolódnak. Ha egy másik shard minden rendelés tárolta, azok kellene (egyetlen adatelemet visszaadó lekérdezések) pont lekérdezések nagy számú elvégzésével külön-külön lehet lekérni. Az alábbi ábrán láthatja a szekvenciális beállítása (tartományok) az adatok tárolása a shard.

   ![2. ábra – szilánkok szekvenciális beállítása (tartományok) az adatok tárolása](./_images/sharding-sequential-sets.png)

Ebben a példában a shard kulcsa, mint a legjelentősebb elem, amelyeket az a sorrend idő a rendelés hónapot tartalmazó összetett kulcsot. Rendelések adatok természetes van rendezve, amikor új rendelések létrehozása és egy shard hozzáadni. Néhány tárolók támogatási kétrészes shard kulcsokat tartalmazó partíció fő eleme, amely azonosítja a shard és egy sor kulcsot, amely egyedileg azonosít egy elem található a szilánkcímtárban. Adatok sor a shard kulcs sorrend általában használatban van. Lekérdezések vonatkoznak, és csoportosíthatók kell elemekre billentyűvel a shard a partíciós kulcs ugyanazt az értéket, de a sorkulcs egy egyedi értéket.

**A kivonatoló stratégia**. Ezt a stratégiát célja az esélye, interaktív (szilánkok terhelés le aránytalanul nagy mennyiségű fogadó) csökkentése érdekében. Az adatok elosztja a szilánkok oly módon, hogy éri el az egyes shard méretét és az átlagos terhelés, amely minden shard fog történni. A horizontális logika kiszámítja elemének tárolására a shard kivonatát, egy vagy több attribútumai alapján. A választott kivonatolási függvény kell szét adatok egyenletesen a szilánkok valószínűleg azokat a számítási néhány véletlenszerű elem bevezetésével. Az alábbi ábrán láthatja a horizontális bérlői adatokat egy kivonatoló azonosítók bérlő alapján.

   ![3. ábra – horizontális bérlői adatokat egy kivonatoló azonosítók bérlő alapján](./_images/sharding-data-hash.png)

Tudni, hogy az az előnye, hogy a kivonat stratégia keresztül más horizontális stratégiák, fontolja meg, hogyan egy több-bérlős alkalmazás, amely új bérlők egymás után előfordulhat, hogy hozzárendelése a bérlők szilánkok szerepel az adattárban. Ha a tartomány stratégia, az adatokat a bérlők számára 1-n használata az összes tárolódnak shard A, az adatokat a bérlők n + 1 m összes tárolódnak a shard B, és így tovább. Ha a közelmúltban regisztrált bérlők is a legaktívabb, a legtöbb tevékenység szilánkok, így elérési pontokhoz való kis számú történik. Ezzel szemben a kivonat stratégia foglal le bérlők számára szilánkok alapján kivonatát, a bérlő azonosítója. Ez azt jelenti, hogy a soros bérlők várhatóan osztható ki másik szilánkok, amely lesz a terhelés szétosztását őket. Az előző ábra ezt mutatja be a bérlők számára 55 és 56.

A három horizontális stratégiák rendelkezik, a következő előnyöket és szempontok:

- **Keresési**. Ez kínál a úgy, hogy szilánkok konfigurálható és használható teljesebb körű vezérlése. Virtuális szilánkok használata csökkenti a hatása, ha újraelosztás adatok, mert az új fizikai partíciók felveheti a terhelés kiegyenlítése érdekében. A virtuális shard és a fizikai partíciók, amelyek megvalósítják a szilánkok leképezése nem befolyásolja az alkalmazáskódban shard kulcsot tárolásához és lekéréséhez adatokat használó lehet módosítani. A shard helyek keresésekor adhat egy többletterhelést.

- **Tartomány**. Ez könnyű, és jól működik lekérdezések mert azok több adatelemek gyakran beolvasása a egy egyetlen shard egyetlen műveletben. Ezt a stratégiát adatok könnyebb kezelést biztosít. Például ha ugyanabban a régióban lévő felhasználók ugyanazt a shard, frissítések az egyes időzónában, a helyi terhelés és igény szerint minta alapján ütemezhető. Ezt a stratégiát azonban nem biztosít, optimális terheléselosztás szilánkok között. Szilánkok újraelosztás nehéz feladat, és nem a probléma megoldására egyenetlen terhelés szomszédos shard kulcsok a legtöbb tevékenység esetén.

- **Kivonatoló**. Ezt a stratégiát nyújt további még adatok és a terjesztési jobb esélyét. Irányítása a kivonatoló függvényt segítségével közvetlenül is elvégezhető. Nincs szükség a térkép fenntartásához. Vegye figyelembe, hogy a kivonat számítástechnikai előfordulhat, hogy ugyanazok egy többletterhelést. Szilánkok újraelosztás is nehézkes.

Leggyakoribb horizontális rendszerek valósítják meg a fent ismertetett módszerek valamelyikét, de azt is figyelembe kell venni az alkalmazások üzleti követelmények és a használati adatok mintáinak. Például egy több-bérlős alkalmazásban:

- Alkalmazások és szolgáltatások alapján részekre bonthatók az adatok is. Külön szilánkok kevéssé permanens bérlők esetén sikerült elkülönítse az adatait. Más bérlők engedélyezik a hozzáférést a sebessége Emiatt előfordulhat, hogy javítani.

- A bérlő helye alapján részekre bonthatók az adatok is. Végre az adatokat a bérlők számára egy adott földrajzi régióban offline biztonsági mentésre és karbantartásra csúcsidőn az adott régióban, míg más bérlők számára az adatok online és elérhető a munkaidő alatt marad.

- Nagy értékű bérlők rendelhető a saját személyes, magas hajt végre, mivel alacsonyabb értékű bérlők várható megosztása több sűrűn csomagolt, foglalt szilánkok enyhén betöltve a szilánkok.

- Adatok elkülönítése és adatvédelmi magas fokú igénylő bérlők számára az adatok tárolhatók egy teljesen különálló kiszolgálón.

## <a name="scaling-and-data-movement-operations"></a>Méretezés és az adatok mozgási műveletek

A horizontális stratégiák mindegyikének azt jelenti, különböző képességeket, és a skála, kibővítési adatátvitel kezelése, és fenntartásáért állapot összetettségi szintek.

A keresési stratégia lehetővé teszi a méretezés és az adatok mozgási műveletek végzett felhasználói szinten, online vagy offline állapotú. A módszer akkor felfüggesztése néhány vagy minden olyan felhasználói tevékenység (például során csúcsidőn kívüli időszakok), helyezze át az adatokat az új virtuális partíciót és a fizikai horizontális skálázás, a leképezések módosításához, érvénytelenné válnak vagy frissítse a gyorsítótárak, hogy ezek az adatok tárolásához, majd engedélyezi a felhasználói tevékenység folytatása. Gyakran ilyen művelet központilag kezelhetők. A keresési stratégia magas kérelmeznék állapotot és a replika rövid igényel.

A tartomány stratégia méretezés és az adatok adatátviteli műveletek, amelyek általában kell végrehajtani, ha egy részét vagy egészét az adattár offline állapotban, mert kell felosztása és egyesíti a szilánkok keresztül az adatok néhány korlátozást ír elő. Az adatok szilánkok egyensúlyba áthelyezése nem a probléma megoldására egyenetlen terhelés szomszédos shard kulcsok vagy az azonos tartományba adatokat azonosítókat a legtöbb tevékenység esetén. A tartomány stratégia is szükség lehet néhány állapotban ahhoz, hogy a tartomány leképezése a fizikai partíciók fenntartásához.

A kivonatoló stratégia teszi méretezés és az adatok mozgási műveletek összetettebb, mivel a partíciós kulcsok kivonatokat a shard kulcsok vagy az adatok azonosítók. Minden shard új helyének meg kell határozni a kivonatoló függvényt a, vagy adja meg a megfelelő leképezések módosítani a függvény. A kivonatoló stratégia azonban nincs szükség szerinti karbantartási.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- Horizontális skálázási értéke más típusú particionálás, például a függőleges particionálásból, és működési particionálás kiegészítő. Például egyetlen shard függőleges particionálva entitásokat is tartalmazhat, és egy működő partíció is implementálható több szegmensben osztják. Partícionálásra vonatkozó további információkért lásd: a [adatok particionálása útmutatást](https://msdn.microsoft.com/library/dn589795.aspx).

- Elosztott terhelésű, így azok minden kezelni hasonló mennyiségű i/o szilánkok tartsa. Adatokat beszúrni, törölni, mivel fontos rendszeresen egyensúlyba a szilánkok garantálja az még akkor is, terjesztési, és csökkenti annak esélyét, a csatlakozási pontokhoz. Újraelosztás lehet egy drága művelet. A szükségességét újraelosztás csökkentéséhez tervezze meg a növekedési biztosításával, hogy minden egyes shard tartalmazza-e elegendő szabad terület a várt kötet változások kezelésére. Akkor is kell kidolgozni, azokat a stratégiákat és parancsfájlok segítségével gyorsan egyensúlyba szilánkok, ha erre akkor van szükség.

- A szilánkok kulcs stabil adatokat használjon. Ha megváltozik a shard kulcsot, a megfelelő adatelem előfordulhat között szilánkok, a frissítési műveletek által végzett munka mennyiségének növelését. Ezért ne a shard kulcs alapozva potenciálisan "volatile" információkat. Ehelyett attribútumokat, amelyek állandó keresse meg, vagy természetes alkotnak, amely egy kulcsot.

- Győződjön meg arról, hogy a shard kulcsok egyediek-e. Például ne autoincrementing mezőkkel shard kulcsként. Egyes rendszerek, a autoincremented mező nem lehet koordinált szilánkok, valószínűleg eredményezve ugyanazt a shard kulcsa eltérő szilánkok elemek között.

    >  Más mezők, amelyek nincsenek shard kulcsok Autoincremented értékeinek hibákat is okozhat. Például ha autoincremented mezők létrehozásához használt egyedi azonosítót, majd két különböző elem található különböző szilánkok hozzárendelhető ugyanezzel az azonosítóval.

- Nem lehet megtervezésére shard kulcsa megegyezik a követelmények minden lehetséges lekérdezés az adatok alapján lehetséges. Az adatok a legtöbb gyakran végre shard lekérdezése, és szükség esetén hozzon létre másodlagos index táblákat támogatja az, hogy a feltétel alapján attribútumokat, amelyek nem részei a shard kulcs használatával adatainak beolvasása. További információkért lásd: a [Index táblázat mintát](index-table.md).

- Csak egyetlen shard elérő lekérdezésekben, hatékonyabbak, mint adatainak lekérése több szegmensben osztják, így csak a végrehajtási egy horizontális skálázási rendszer, amely számos különböző szilánkok-ban tárolt adatok lekérdezés alkalmazások eredményez. Ne feledje, hogy egyetlen shard entitások többféle típusú adatok is tartalmazhat. Ugyanazt a shard alkalmazás hajtja végre külön olvasások számának csökkentése érdekében az adatokat, hogy a kapcsolódó entitásokból, amely általában a rendszer megkérdezi együtt (például az ügyfelek és az azok elhelyezett rendeléseket részleteit) megtartása denormalizing figyelembe kell venni.

    >  Ha egy shard az entitás egy másik shard tárolt entitás hivatkozik, tartalmazza a shard kulcsot az első entitás séma részeként a második entitás. Ez segít szilánkok keresztül kapcsolódó adatokra hivatkozó lekérdezések teljesítményének javítása érdekében.

- Ha egy alkalmazás adatainak lekérése több szilánkok lekérdezések kell végrehajtani, esetleg az adatok beolvasása a párhuzamos tevékenységek. Például szétterítési lekérdezések, ahol több szilánkok adatait párhuzamosan beolvasni, és egyetlen eredmény majd összesítése. Azonban ez a megközelítés mindenképpen ad hozzá néhány összetettsége a megoldás az adatok hozzáférési logikai.

- Számos alkalmazás létrehozásakor kis szilánkok nagyobb számú lehet hatékonyabb, mint használjon nagy szilánkok kis mennyiségű, mert nagyobb teret hagynak a terheléselosztást is biztosítanak. Ez is lehet hasznos, ha várhatóan szilánkok áttelepítését egyik fizikai helyről a másikra szeretné. Helyezze át egy kis shard gyorsabb, mint a nagy egy áthelyezését.

- Ellenőrizze, hogy minden shard tárhely csomópontot számára elérhető erőforrások elegendőek a méretezhetőségi követelményeinek adatméret és átviteli sebesség kezelésére. További információkért lásd: "Tervezése partíciók a méretezhetőség" szakasz a a [adatok particionálása útmutatást](https://msdn.microsoft.com/library/dn589795.aspx).

- Vegye figyelembe, hogy minden szilánkok referenciaadatok replikálása. Ha egy művelet kiolvassa az adatokat a shard statikusak vagy lassan adatokat is hivatkozik ugyanabban a lekérdezésben részeként, adatok hozzáadása a szilánkcímtárban. Az alkalmazás is majd beolvasni az összes adat a lekérdezés könnyen, anélkül, hogy egy további oda-vissza legyen egy külön adattároló.

    >  Ha a referenciaadatok több szegmensben osztják megváltozik, a rendszer minden szilánkok ezeket a módosításokat kell szinkronizálnia. A rendszer inkonzisztenciát fokú tapasztalhat, a szinkronizálásra kerül sor. Ha így tesz, akkor tervezzen a tudja kezelni az alkalmazásokat.

- A hivatkozási integritás-és szilánkok, így, csökkentse minimálisra több szilánkok adatokat érintő műveletek közötti konzisztencia nehézkes lehet. Ha egy alkalmazás közötti szilánkok adatok módosítania kell, értékelje ki a teljes adatkonzisztencia ténylegesen szükséges-e. Ehelyett egy általánosan használt megközelítés a felhőben, hogy alkalmazza a végleges konzisztencia. Mindegyik partíció adatait a külön-külön frissül, és az alkalmazáslogikát kell venniük biztosításáért, hogy a frissítések összes végrehajtása sikeres legyen, valamint lekérdezése az adatok egy idővel konzisztenssé során felmerülő inkonzisztenciák kezelése a művelet fut. A végleges konzisztencia kapcsolatos további információkért lásd: a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).

- Konfigurálása és kezelése a szilánkok sok bonyolult lehet. Feladatok, például a figyelés, biztonsági mentése, konzisztencia-ellenőrzés és naplózás vagy naplózás jellegétől több szegmensben osztják és-kiszolgálók, valószínűleg tárolt több helyen. Ezek a feladatok valószínűleg parancsprogramokkal vagy más automatizálási megoldásokat kell végrehajtani, de, előfordulhat, hogy nem szünteti meg teljesen a további felügyeleti követelményeket.

- Szilánkok geolocated lehet, hogy a bennük lévő adatok mérete megközelítőleg, az azt használó alkalmazás példányait. Ez a megközelítés jelentősen javíthatja a teljesítményt, de más-más helyen több szegmensben osztják el kell érnie feladatokhoz több figyelmet igényel.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Használja ezt a mintát, ha a adattárat valószínű, hogy szükség van egy egyetlen tárolási csomópont számára elérhető erőforrások felüli vagy csökkenti a tárolóban a versengés a teljesítmény javítása érdekében.

>  A horizontális elsősorban a jobb teljesítmény és méretezhetőség rendszert, de módszer továbbá akkor is tovább fejlesztheti rendelkezésre állási módját az adatokat külön partíciókra oszlik miatt. Egy partíció hibája miatt nem feltétlenül megakadályozhatja, hogy egy alkalmazás nem férhet hozzá a többi partíció-ban tárolt adatok, és egy olyan operátort hajthat végre karbantartás vagy egy vagy több partíció helyreállítását anélkül, hogy az alkalmazás teljes adatai nem érhetők el. További információkért lásd: a [adatok particionálása útmutatást](https://msdn.microsoft.com/library/dn589795.aspx).

## <a name="example"></a>Példa

A következő példa a C# használ az SQL Server-adatbázisok szilánkok működött. Minden adatbázis tárolja az alkalmazások által használt adatok egy részét. Az alkalmazás lekéri az adatokat, amelyek segítségével a saját horizontális programot (Itt látható egy példa a kívánt szétterítési lekérdezések) szilánkok között van elosztva. Az egyes shard található adatok részleteit a hívott metódus által visszaadott `GetShards`. Ez a módszer egy enumerálható listáját adja vissza `ShardInformation` objektumok, ahol a `ShardInformation` típus minden shard és az SQL Server kapcsolati karakterlánc egy alkalmazás kell használni (a kapcsolati karakterláncok nem látható a shard csatlakozni azonosítót tartalmaz. a kódpéldát).

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

Az alábbi kód bemutatja, hogyan az alkalmazás használja listáját `ShardInformation` objektumok kér le adatokat az egyes shard párhuzamos lekérdezés végrehajtásához. A részletek a lekérdezés nem látható, de ebben a példában a beolvasott adatok egy karakterláncot tartalmaz, amely sikerült egy ügyfél például a név adatok tárolásához, ha a szilánkok ügyfelek részleteit tartalmazza. Az eredmények összesítése egy `ConcurrentBag` gyűjtemény az alkalmazás által a feldolgozás céljából.

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

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:
- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). Biztosítja az egységességet az adatok különböző szilánkok pontjain szükség lehet. Összesíti a kérdésekkel karbantartása konzisztencia elosztott adatokon keresztül, és ismerteti az előnyöket és a különböző konzisztencia modellek kompromisszumot.
- [Útmutatás particionálás adatok](https://msdn.microsoft.com/library/dn589795.aspx). A további problémák széles horizontális adattárat vethet fel. A méretezhetőség javítása, versengés csökkentése, és a teljesítmény optimalizálása érdekében a felhőben adattárolókhoz particionálás viszonyítva problémákat ismerteti.
- [Index táblázat mintát](index-table.md). Néha nem lehet teljes mértékben támogatja az csak a shard kulcs kialakításának keresztül. Lehetővé teszi az alkalmazás gyorsan adatainak lekérése nagy adattároló eltérő a shard kulcs kulcs megadásával.
- [Materializált nézet mintát](materialized-view.md). Az egyes lekérdezési műveletek teljesítményének fenntartása érdekében célszerű összesítése és adatösszegzés, materializált nézetek létrehozására, különösen akkor, ha az összegzett adatokat a szilánkok között van elosztva információkat alapul. Ismerteti, hogyan hozhat létre, és ezek a nézetek feltöltéséhez.
- [A shard során tapasztalatokat](http://www.addsimplicity.com/adding_simplicity_an_engi/2008/08/shard-lessons.html) az egyszerűség kedvéért hozzáadása blogjában.
- [Adatbázis-horizontális](http://dbshards.com/database-sharding/) CodeFutures webhelyén.
- [Méretezhetőség stratégiák ismertetése: Adatbázis horizontális](http://blog.maxindelicato.com/2008/12/scalability-strategies-primer-database-sharding.html) maximális Indelicato blogjában.
- [Épület méretezhető adatbázisok: Informatikai szakemberek és hátrányának a különböző adatbázis horizontális programok](http://www.25hoursaday.com/weblog/2009/01/16/BuildingScalableDatabasesProsAndConsOfVariousDatabaseShardingSchemes.aspx) Dare Obasanjo blogjában.
