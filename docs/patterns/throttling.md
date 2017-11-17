---
title: "Szabályozás"
description: "Egy alkalmazás, egy adott bérlő vagy az egész szolgáltatás egy példánya által használt erőforrások fogyasztásának szabályozzák."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- performance-scalability
ms.openlocfilehash: 29156fc72f40a952dd53adcb20ffa7c3d0af79b4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="throttling-pattern"></a>Sávszélesség-szabályozási minta

[!INCLUDE [header](../_includes/header.md)]

Egy alkalmazás, egy adott bérlő vagy az egész szolgáltatás egy példánya által használt erőforrások fogyasztásának szabályozzák. Engedélyezheti, hogy a rendszer továbbra is működik, és megfelelnek a szolgáltatásiszint-szerződések, még akkor is, ha az igény szerinti növelését egy rendkívüli betöltése az erőforrás.

## <a name="context-and-problem"></a>A környezetben, és probléma

A felhőalapú alkalmazások terhelése jellemzően az aktív felhasználóknak a számát, vagy azok hajtja végre műveletek alapján időnként eltérő. Például további felhasználók valószínűleg aktív munkaidőben, vagy hajtsa végre a számítástechnikai, olcsóbbá analytics minden hónap végén szükséges lehet a rendszer. Is előfordulhat hirtelen és a nem várt felszakadásáig tevékenységben. Ha a rendszer ügyféloldali bővítmények feldolgozási követelményeivel meghaladja a rendelkezésre álló erőforrások kapacitása, lesz érinti a gyenge teljesítményt, és akkor is sikertelen lehet. Ha a rendszer egy egyeztetett szolgáltatási szintjének kielégítéséhez, ilyen hiba elfogadhatatlan lehet.

Sok stratégiák még elérhető a felhőben, attól függően, hogy az üzleti célokat, az alkalmazás különböző terhelés kezelésére. Egy stratégia az automatikus skálázás használandó felel meg a felhasználó a kiosztott erőforrásokat kell egy adott időpontban. Ez magában hordozza a felhasználói igényeknek, miközben optimalizálja a futó költségek következetesen teljesítéséhez. Azonban amíg automatikus skálázás is elindíthatja a további erőforrások kiépítése, azonnali a kiépítés nem. Igény szerinti gyorsan növekszik, ha az időt egy ablak lehet erőforrás hiány esetén.

## <a name="solution"></a>Megoldás

Egy alternatív stratégia, amelynek az automatikus skálázást, hogy lehetővé teszik az alkalmazások csak legfeljebb erőforrásainak használatához, és majd azokat szabályozás, ha eléri ezt a korlátot. A rendszer hogyan használja erőforrásokat, hogy ha használat meghaladja a küszöbértéket, képes szabályozni a egy vagy több felhasználók által érkező kérések célszerű figyelemmel kísérni. Ezzel a lépéssel engedélyezi a rendszer folytatja a működését, és megfelelnek a szolgáltatásszint-szerződések (SLA) vannak érvényben. Erőforrás-használat figyelésével kapcsolatos további információkért lásd: a [Instrumentation és Telemetriai útmutatást](https://msdn.microsoft.com/library/dn589775.aspx).

A rendszer több sávszélesség-szabályozási stratégiák, beleértve a sikerült megvalósításához:

- Egy adott felhasználóhoz, aki már érkező kérelmeket visszautasítja a rendszer API-k másodpercenként több mint n alkalommal keresztül elérhető egy adott időszakban. Ehhez a rendszer minden egyes bérlő vagy az alkalmazást futtató felhasználók használt erőforrások mérni szeretné. További információkért lásd: a [mérési útmutató szolgáltatásához](https://msdn.microsoft.com/library/dn589796.aspx).

- Letiltása, vagy kijelölt nem szükséges szolgáltatások funkcióinak terhelése, hogy fontos szolgáltatások akadálymentesnek elegendő erőforrással rendelkező futtathatók. Például ha az alkalmazás az adatfolyam-videokimenetéhez, azt átállítása volt alacsonyabb felbontása.

- Használatával Terheléskiegyenlítés sima tevékenység mennyiségét (Ez a megközelítés által további részletesen is ismertetjük a [várólista alapú betöltése simítás mintát](queue-based-load-leveling.md)). Ezt a módszert használja egy több-bérlős környezetben minden bérlő számára a teljesítmény csökkenti. Ha a rendszernek támogatnia kell a bérlők a különböző SLA kombinációját, a munka nagy értékű bérlők azonnal is végrehajthatók. Más bérlők kérelmek vissza tartott, és kezelésének, amikor a várakozó megkönnyítését rendelkezik. A [prioritású várólistára mintát][] segítségével ennek a végrehajtására használható.

- A késleltető alacsonyabb prioritású alkalmazások vagy a bérlők nevében végrehajtott műveletek. Ezek a műveletek felfüggesztve, vagy korlátozott, a bérlő tájékoztatja, hogy a rendszer elfoglalt, és hogy a művelet később meg kell ismételni generált kivétel miatt.

Az ábrán látható erőforrás-használat (a memória, Processzor, sávszélesség és más tényezők kombinációja) tartozó területgrafikon idő függvényében, az alkalmazásokat, amelyek létesített három funkcióját használja. Egy olyan szolgáltatása, funkciókat, például a feladatok, egy bonyolult számításhoz megvalósító kódot, vagy olyan elem, például egy memórián belüli gyorsítótárral szolgáltatást nyújt egy meghatározott elvégző összetevőnek területe. Ezek a funkciók tartalma A, B és c kiszolgálóra.

![1. ábra – erőforrás-használat három felhasználók nevében futó alkalmazások idő függvényében ábrázoló grafikon](./_images/throttling-resource-utilization.png)


> Közvetlenül egy adott szolgáltatáshoz sor alatt azt jelzi, ha ez a szolgáltatás hivatkoznak alkalmazások által használt erőforrások. Például alatt használt vonal szolgáltatása mutat be arról alkalmazások által használt erőforrások között a funkció A sorokat és a szolgáltatás B és a terület azt jelzi, a szolgáltatás b összesítése meghívása alkalmazások által használt erőforrások funkció a területet használja az egyes szolgáltatásokhoz jeleníti meg a rendszer a teljes erőforrás-használatát.

Az előző ábrán láthatja a eredő késleltető a műveletek. Közvetlenül megelőző helyreállítási pontot T1 idő a teljes erőforrások vannak rendelve az összes alkalmazást használja ezeket a funkciókat elérni a küszöbérték (erőforrás-használati korlátja). Ezen a ponton az alkalmazások szerepelnek, az elérhető erőforrások kimerítsék veszélye. Ebben a rendszerben a szolgáltatás a B kiszolgáló lesz kevésbé létfontosságú, mint A szolgáltatás vagy funkció C, ezért átmenetileg le van tiltva, és az erőforrásokat, amelyek az általa kiadott. A T1 és T2, a szolgáltatás A és a szolgáltatás C használó alkalmazások továbbra is időpont között futtató normál. Végül, az erőforrás-e két szolgáltatás használatát a pontra mérsékli, amikor T2 időpontban nincs elegendő kapacitással a szolgáltatás B ismét engedélyeznie.

Az automatikus skálázás és a szabályozás megközelítések is kombinálható védheti az alkalmazások gyorsabban kezelhetővé és SLA-k belül. Ha az igény szerinti várhatóan magas, sávszélesség-szabályozás ideiglenes megoldást kínál közben a rendszer kimenő méretezi. Ezen a ponton visszaállítása végezhető el a rendszer az összes funkcióját.

Az alábbi ábrán jeleníti meg, a teljes erőforrás-használat területgrafikon idő függvényében a rendszerben futó összes alkalmazást, és bemutatja, hogyan sávszélesség-szabályozás kombinálható az automatikus skálázást.

![2. ábra – diagramját eredő egyesítésével az automatikus skálázás szabályozása](./_images/throttling-autoscaling.png)


A T1 időpontban a megadásával az erőforrás-használata enyhe legfeljebb küszöbértéket. A rendszer ezen a ponton, megkezdheti a kiterjesztése. Ha az új erőforrások nem lesznek rendelkezésre elég gyorsan, majd előfordulhat, hogy felhasználhatók, a meglévő erőforrásokat, és sikertelen lehet, a rendszer. A probléma elkerüléséhez, hogy a rendszer átmenetileg folyamatban van, a fentebb leírt módon. Ha automatikus skálázás befejeződött, és a további erőforrások elérhetők, a sávszélesség-szabályozás enyhíteni lehet.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:

- Sávszélesség-szabályozás egy alkalmazást, és a stratégia használatához-e egy architekturális döntés, amely hatással van a rendszer az egész kialakítása. Sávszélesség-szabályozás figyelembe kell venni az alkalmazás tervezési folyamat korai szakaszaiban nem egyszerű hozzáadása után a rendszer hajtották végre, mert.

- Sávszélesség-szabályozás kell elvégezni gyorsan. A rendszer legyen képes növelni a tevékenységek észlelésére, és ennek megfelelően reagálni. A rendszer is állíthatja vissza az eredeti állapotba gyorsan után a terhelés rendelkezik megkönnyítését kell lennie. Ehhez az szükséges, hogy a megfelelő teljesítményadatokat folyamatosan rögzített, és figyeli.

- Ha a szolgáltatás ideiglenesen megtagadja az egy felhasználói kérelem, az egy adott hibakódot, az ügyfélalkalmazás tisztában van azzal, hogy a művelet végrehajtásához az elutasítás oka, hogy szabályozás miatt kell visszaadnia. Az ügyfélalkalmazás is Várjon egy ideig, mielőtt megpróbálná megismételni a kérelmet.

- Sávszélesség-szabályozás során a rendszer autoscales ideiglenes intézkedésként is használható. Bizonyos esetekben célszerűbb egyszerűen szabályozás, nem pedig válhatnak Ha egy tevékenység kapacitásnövelés hirtelen, és nem várhatóan hosszabb élettartamú, mivel a skálázás adhat hozzá jelentősen költségeket.

- Ha sávszélesség-szabályozás használatos során a rendszer autoscales ideiglenes mérték, és az erőforrás iránti igények kielégítése érdekében nagyon gyorsan növekszik, ha a rendszer nem feltétlenül működik-e a folytatáshoz&mdash;még akkor is, ha a szabályozottan halmozott módban működik. Ha ez nem elfogadható, fontolja meg a nagyobb kapacitás tartalék karbantartásáért, valamint a szigorúbb automatikus skálázás konfigurálása.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta használata:

- Annak érdekében, hogy a rendszer továbbra is megfelel a szolgáltatási szintek.

- Megakadályozhatja, hogy egyetlen bérlő a legaktívabbak egy alkalmazás által biztosított erőforrások.

- Kezelendő felszakadásáig tevékenységben.

- Optimalizálásához költség-a rendszer a maximális erőforrás szintek korlátozásával szükséges biztosítható, hogy működik-e.

## <a name="example"></a>Példa

A végső ábra bemutatja, hogyan sávszélesség-szabályozás megvalósítható a több-bérlős rendszerek. Az egyes a bérlő szervezet felhasználók férhetnek hozzá a felhőben üzemeltetett alkalmazás, ahonnan töltse ki és felmérések elküldeni. Az alkalmazás, amely figyeli a sebesség, amellyel a felhasználók az alkalmazásnak küldött kérelmek elküldése instrumentation tartalmazza.

Leállítja, nehogy válaszidejét és más felhasználók számára az alkalmazás rendelkezésre állásának érintő egy bérlő felhasználóit, a korlát vonatkozik a panelhez egy felhasználói beküldhetik kérelmek másodpercenkénti száma. Az alkalmazás blokkolja az e korláton kérelmek.

![3. ábra - végrehajtási a sávszélesség-szabályozás egy több-bérlős alkalmazásban](./_images/throttling-multi-tenant.png)


## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is megfelelő ebben a mintában végrehajtása során:
- [Telemetria útmutató és Instrumentation](https://msdn.microsoft.com/library/dn589775.aspx). Sávszélesség-szabályozás attól függ, hogyan fokozottan használatban van egy szolgáltatás adatokat gyűjt. Ismerteti, hogyan hozhat létre, és egyéni figyelési információkat.
- [Útmutatás mérési szolgáltatás](https://msdn.microsoft.com/library/dn589796.aspx). Szolgáltatások használatának mérési felmérheti, azok használata érdekében ismerteti. Ez az információ akkor lehet hasznos az adatátviteli szolgáltatás módjának meghatározása.
- [Automatikus skálázás útmutatást](https://msdn.microsoft.com/library/dn589774.aspx). Sávszélesség-szabályozás belső használatra során a rendszer autoscales, vagy távolítsa el a rendszer automatikus skálázásra szükségességét is használható. Automatikus skálázás stratégiák információkat tartalmaz.
- [Betöltési simítás mintát várólista alapú](queue-based-load-leveling.md). Várólista alapú terhelés simítás, akkor egy gyakran használt eszköz sávszélesség-szabályozás megvalósításához. A várólista, mely akkor is igaz, a sebesség, amellyel az alkalmazás által küldött kérések érkeznek, egy kimenő puffer működhet.
- [prioritású várólistára mintát][]. A rendszer a sávszélesség-szabályozási stratégia részeként queuing prioritás segítségével fenntartható teljesítmény kritikus vagy magasabb érték alkalmazások, miközben csökkenti a kevésbé fontos alkalmazások teljesítményét.

[prioritású várólistára mintát]: priority-queue.md