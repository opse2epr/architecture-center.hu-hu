---
title: A Feladatütemező ügynök felügyelő
description: Egy készletét műveletek között elosztott készlete szolgáltatások és az egyéb távoli erőforrásokhoz.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- resiliency
ms.openlocfilehash: 03bfe2fe96b3b81d547cfedb075bcf855846b668
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="scheduler-agent-supervisor-pattern"></a>A Feladatütemező ügynök felügyelő minta

[!INCLUDE [header](../_includes/header.md)]

Egy készletét elosztott műveletek egyetlen műveletként. Ha a műveletek sikertelenek, próbálja meg transzparens módon kezeli a hibák, vagy pedig a munkahelyi végeznie, így a teljes művelet sikeres vagy sikertelen egész visszavonása. Rugalmasság ez hozzáadása egy elosztott rendszert, lehetővé téve számára a helyre, majd próbálja megismételni a végrehajtandó műveleteket átmeneti kivételek, hosszú ideig tart és folyamat hibái miatt sikertelen.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy alkalmazás feladatokat, amelyek közé tartozik a száma, amelyeket lehet, hogy távoli szolgáltatások invoke vagy távoli erőforrások eléréséhez lépéseiért hajt végre. Előfordulhat, hogy az egyes lépéseket egymástól független, de azok vannak összehangolva. az alkalmazás logika, amely megvalósítja a feladatot.

Amikor csak lehetséges, az alkalmazás győződjön meg arról, hogy a feladat befejezését fut, és hárítsa el a távoli szolgáltatások vagy az erőforrásokhoz való hozzáférés során felmerülő esetleges hibákat. Hiba történik a számos oka lehet. Például a hálózat esetleg nem működik, kommunikációs sikerült lehet megszakítani, elképzelhető, hogy a távoli szolgáltatás nem válaszol vagy instabil állapotban vagy távoli az erőforrás átmenetileg nem érhetők el, lehet, hogy az erőforrás-korlátozások miatt előfordulhat, hogy lehet. Sok esetben a hibák átmenetiek lehetnek, és segítségével kezelhetik a [újrapróbálkozási mintát][retry-pattern].

Ha az alkalmazás állandó hibát észlel, nem könnyen helyreállíthatók, állítsa helyre a rendszer konzisztens állapotba, és a teljes műveletet integritását képesnek kell lennie.

## <a name="solution"></a>Megoldás

A Feladatütemező ügynök felügyelő mintát a következő szereplője határozza meg. Ezek szereplője levezényelni a teljes feladat részeként elvégzendő lépések.

- A **Feladatütemező** rendezi a lépéseket, amelyek a feladat futtatását és koordinálja a műveletet. Ezeket a lépéseket egy folyamat vagy munkafolyamat lehet egyesíteni. Az ütemező felelős győződjön meg arról, hogy a munkafolyamat utasításait a megfelelő sorrendben történik. Mivel minden egyes lépést, a Feladatütemező rögzíti a munkafolyamat, például a "lépés a még nem indult el," "lépés fut" állapotba, vagy a "lépése befejeződött." Az állapot is tartalmaznia kell a lépés befejezéséhez engedélyezett idő egy felső korlátot, a befejezéséhez által idő hívása. Ha egy lépés szükség van a távoli szolgáltatás vagy az erőforrás elérésére, a Feladatütemező hív meg, a megfelelő ügynököt, és átadja azt a végzett munka részleteit. Az ütemező általában egy ügynök aszinkron kérelem/válasz messaging szolgáltatással kommunikál. Ennek segítségével valósítható várólisták, de más elosztott üzenetküldő technológiák volt használható, helyette.

    > Az ütemező egy hasonló funkciót a folyamat Manager a hajt végre a [folyamat Manager mintát](http://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html). A tényleges munkafolyamat általában definiálva, és egy munkafolyamat-motor, amely vezérli az ütemező által megvalósított. Ez a megközelítés a munkafolyamatot az ütemező üzleti logikája leválasztja.

- A **ügynök** logika, amely magában foglalja a távoli szolgáltatás hívása, vagy egy távoli lépése, a feladatütemezés által hivatkozott erőforráshoz való hozzáférés tartalmazza. Minden ügynök általában becsomagolja a egy egyetlen szolgáltatás vagy az erőforrás, hívások végrehajtási a megfelelő hiba kezelését, és próbálkozzon újra a programot a (függvényében időtúllépés megkötés, a későbbiekben olvashat). A lépéseket a munkafolyamatban az ütemező által futtatott használja más lépéseket több szolgáltatásokhoz és erőforrásokhoz, ha minden lépés (Ez a minta egy megvalósítási részletei) egy másik ügynök esetleg hivatkoznak.

- A **felügyelő** a lépéseket a az ütemező által végzett feladat állapotát figyeli. Rendszeres időközönként fut (a lesz adott rendszer), és megvizsgálja az ütemező által fenntartott lépéseket állapotát. Ha azt észleli, hogy azokat, lejárt, vagy sikertelen volt, rendszerezi helyreállítani a lépés, vagy hajtsa végre a megfelelő javító intézkedés (Ez lehet, hogy tartalmaz, amely egy lépés állapotának módosítása) a megfelelő ügynök. Vegye figyelembe, hogy a helyreállítási vagy a javító műveleteket az ütemező és az ügynökök alapján történik. A felügyelő egyszerűen igényeljen, hogy ezeket a műveleteket kell elvégezni.

A Feladatütemező, az ügynök és a felügyelő összetevői logikai és fizikai végrehajtásuk használt technológiától függ. Több logikai ügynökök például egyetlen webszolgáltatás részeként kell végrehajtani.

A Feladatütemező a feladat és a tartós tárolóban, az állapot tárolásának nevű egyes lépéseinek állapotát vonatkozó adatokat tárolja. A felügyelő használhatja ezeket az információkat annak meghatározásához, hogy a lépés nem sikerült. Az ábra azt mutatja be, a Feladatütemező, az ügynökök, a felügyelő és az állapot tárolásának közötti kapcsolat.

![1. ábra – a Feladatütemező ügynök felügyelő mintában actors](./_images/scheduler-agent-supervisor-pattern.png)


> A minta egy egyszerűsített verziója látható. Valós megvalósításában valószínűleg egymással párhuzamosan fut, minden egyes feladatok egy részét az ütemező több példányát. Hasonlóképpen a rendszer futtathat minden ügynök, vagy még több felügyelők több példányát. Ebben az esetben felügyelők kell koordinálja egymással gondosan győződjön meg arról, hogy nem "versenyeznek" a sikertelen lépéseket és feladatokat helyreállítása a munkájukat. A [vezető választás mintát](leader-election.md) biztosít egy lehetséges megoldás erre a problémára.

Ha az alkalmazás készen áll a futtatásra egy feladatot, akkor kérést küld az ütemező. A Feladatütemező rekordok kezdeti állapotinformációkat a feladatot, és lépést (például lépés még nem indult el) a állapot tárolásának, majd elindítja a munkafolyamat által definiált műveletek végrehajtása. Az ütemező indulásakor az egyes lépéseket, akkor frissíti az állapottároló (például lépés fut) az adott lépés állapotára vonatkozó adatokat.

Ha egy lépés hivatkozik a távoli szolgáltatás vagy az erőforrás, a Feladatütemező üzenetet küld a megfelelő ügynök. Az üzenet a szolgáltatáshoz adnak át vagy nem érhető el az erőforrás a művelet befejezéséhez által ideje mellett kell az ügynököt adatokat tartalmazza. Ha az ügynököt a művelet sikeresen befejeződött, a Feladatütemező választ ad vissza. Az ütemező az állapottároló (például lépés befejeződött) állapot frissítése, majd hajtsa végre a következő lépéssel. A folyamat folytatódik, amíg a teljes feladat be nem fejeződik.

Az ügynök valósíthatja meg semmilyen újrapróbálkozási logika, amely a munkájuk elvégzéséhez szükséges. Azonban ha az ügynök nem végrehajtják a tevékenységeket, végezze el által lejárta előtt, a Feladatütemező azt fogja feltételezni, hogy a művelet sikertelen volt. Ebben az esetben az ügynök állítsa le a tevékenységeket és nem próbálja meg bármit térjen vissza az ütemező (még egy hibaüzenet), vagy próbálja bármely formájára vonatkozó helyreállítási. Ez a korlátozás oka, hogy egy lépés túllépte az időkorlátot vagy sikertelen volt, követően az ügynök egy másik példánya ütemezhető a hibás lépés (a műveletet később) futtatásához.

Ha az ügynök nem sikerül, a Feladatütemező nem kapott választ. A minta nem különböztetni a lépést, amely túllépte az időkorlátot, a másik valóban sikertelen volt.

Egy lépés átlépi az időkorlátot vagy meghiúsul, ha az állapottároló egy rekordot, amely azt jelzi, hogy a lépés futtatja, de a befejezéséhez által idő fog átadott fogja tartalmazni. A felügyelő ehhez hasonló lépéseket keresi, és megpróbálja helyreállítani őket. Egy lehetséges stratégia a befejezéséhez által értékét a lépés elvégzése után, és az ütemező azonosítása a lépés túllépte az időkorlátot, majd üzenetküldéshez rendelkezésre álló időt kiterjeszteni a felügyelő szolgál. Az ütemező majd megpróbálja ismételje meg ezt a lépést. Azonban ez a kialakítás megköveteli a feladatokat az idempotent.

A felügyelő megakadályozhatja, hogy megegyezik a hajtva, ha folyamatosan meghibásodik, vagy időtúllépés módosítania kell. Ehhez az szükséges, a felügyelő sikerült karbantartása minden egyes lépést, és az állapotadatokat, az a tárolóban újrapróbálkozások számát. Ha ez a szám eléri az előre definiált a felügyelő is elfogadása előtt, akkor újra kell a lépésben az általános gyakorlat, hogy a tartalék fel fogja oldani az ebben az időszakban az ütemező értesítésére hosszabb ideig vár a. Azt is megteheti, a felügyelő tud üzenetet küldeni az ütemező kéréséhez vonhatók vissza, a teljes tevékenység végrehajtása egy [Compensating tranzakció mintát](compensating-transaction.md). Ez a megközelítés a Feladatütemező és minden egyes lépést, amely a sikeresen befejeződött a kompenzációs műveletek végrehajtásához szükséges adatokat szolgáltató ügynökök függ.

> A Feladatütemező és az ügynökök figyelésére, és indítsa újra őket, ha azt elmulasztják a felügyelő célja nem. A rendszer ezen része az infrastruktúra ezeket az összetevőket futnak kezelje. Hasonlóképpen, a felügyelő lehetnének fut a feladat az ütemező által végzett műveletek a tényleges üzleti ismerete (egyebek között a helyesbítéshez amennyiben ezek a feladatok nem). Ez az a célja az ütemező által megvalósított a munkafolyamat-logikát. A felügyelő kizárólagos felelőssége, hogy határozza meg, hogy a lépés nem sikerült, és meg kell ismételni, vagy a teljes feladat lehet visszavonni a hibás lépést tartalmazó elrendezése.

Az ütemező meghibásodás után újraindul, vagy az ütemező által végzett a munkafolyamat váratlanul leáll, ha az ütemező legyen meg tudja határozni, ha az nem volt kezelése szolgáló aktív feladat állapotát, és készüljön, ez a feladat folytatása ettől a ponttól. A részleteket a folyamat során valószínűleg adott rendszer. A feladat nem állítható helyre, ha a feladat már végzett munka visszavonása szükség lehet. Emellett előfordulhat, hogy végrehajtási egy [tranzakció compensating](compensating-transaction.md).

A fő ebben a mintában előnye, hogy a rendszer váratlan ideiglenes vagy helyreállíthatatlan hiba esetén refs nem. A rendszer kell öngyógyító lehet létrehozni. Például ha egy ügynök vagy a Feladatütemező nem sikerül, egy új elindítható, és a felügyelő rendezheti folytatható egy tevékenység. Ha nem sikerül a felügyelő, egy másik példánya elindítható, és képes átveszik a hiba helyéről. A felügyelő ütemezett rendszeres időközönként, ha egy előre meghatározott időszak után automatikusan indítható új példányt. Az állapot tárolásának replikálható egy még nagyobb fokú rugalmasságot eléréséhez.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:

- Ebben a mintában nehéz lehet megvalósítani, és a rendszer minden lehetséges hibaállapotba alaposan tesztelni.

- Az ütemező által megvalósított helyreállítási/újrapróbálkozási logika összetett és függő állapotadatokat az állapottároló használatban. Is szükség lehet a tartós tárolóban kompenzációs tranzakció végrehajtásához szükséges adatok rögzítéséhez.

- Milyen gyakran a felügyelő fut-e szüksége lesz. Megakadályozhatja, hogy a hibás lépéshez sem blokkolja-e egy alkalmazás hosszú időn keresztül a elég gyakran fusson, de azt nem futtatható úgy, hogy egy terhelés válik.

- A lépéseket egy ügynök által végzett egynél többször is futhat. A programot, amely megvalósítja az alábbi lépéseket az idempotent kell lennie.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Használja ezt a mintát, ha egy folyamat, amely elosztott környezetben, például a felhőben fut, a refs kommunikációs hiba és/vagy a működési probléma kell lennie.

Ez a minta nem feltétlenül alkalmas feladatokat, amelyek nem távoli szolgáltatások meghívása vagy a távoli erőforrásokhoz férnek hozzá.

## <a name="example"></a>Példa

A webes alkalmazás, amely egy kereskedelmi rendszer lett telepítve a Microsoft Azure. Ez az alkalmazás keresse meg a választható termékek és a rendeléseket helyezze futtathatók. A felhasználói felület webes szerepkörben fut, és az alkalmazás rendelés feldolgozása elemei feldolgozói szerepkörök készletként valósíthatók meg. A logika feldolgozási sorrendben részét magában foglalja a távoli szolgáltatás elérésével, és lehet, hogy a rendszer ezen része átmeneti jellegű, vagy további hosszú ideig tart hibák hibalehetőségeket rejt magában. Emiatt a tervezők a Feladatütemező ügynök felügyelő mintát használt feldolgozásakor a rendszer az elemek sorrendje végrehajtásához.

Amikor a vevő egy sorrendet, az alkalmazás egy üzenet, amely leírja a sorrendben, és ez az üzenet egy üzenetsorba küldi hoz létre. Külön küldésének folyamatainak, a feldolgozói szerepkör található üzenet beolvasása, a rendelés részleteit szúr be a rendelések adatbázist, és létrehoz egy rekordot a rendelés folyamat az állapottároló. Vegye figyelembe, hogy a rendelések adatbázist és az állapot tárolásának Beszúrások azonos művelet részeként történik. A folyamat célja, hogy győződjön meg arról, hogy mindkét beszúrása teljes együtt.

A folyamat létrehozza a rendelés állapotinformáció tartalmazza:

- **OrderID**. A rendelések adatbázis sorrendje azonosítója.

- **LockedBy**. A feldolgozói szerepkör kezelése a sorrend Példányazonosítója. Előfordulhat, hogy a feldolgozói szerepkör fut a Feladatütemező aktuális példányai, de minden rendelés csak egy példányban kell kezelnie.

- **CompleteBy**. Az idő a sorrendben kell feldolgozni.

- **ProcessState**. A sorrend kezelése a feladat aktuális állapotát. A lehetséges állapotok a következők:

    - **Függőben lévő**. A sorrend létrejött, de feldolgozása még nem kezdődött.
    - **Feldolgozási**. A megrendelés feldolgozása folyamatban van.
    - **Feldolgozott**. A rendelés feldolgozása sikeresen megtörtént.
    - **Hiba**. Rendelés feldolgozása sikertelen volt.

- **FailureCount**. Hány alkalommal a feldolgozása meghiúsult a sorrend.

Az állapot az információ a `OrderID` mezőt átmásolja az új sorrendbe rendelés azonosítója. A `LockedBy` és `CompleteBy` mezők értéke `null`, a `ProcessState` mező értéke `Pending`, és a `FailureCount` mező értéke 0.

> Ebben a példában a sorrend logika kezelése viszonylag egyszerű, és csak egyetlen lépésben, amely hívja meg a távoli szolgáltatás. Egy összetettebb többlépéses forgatókönyv esetén a folyamat olyan valószínűleg tartalmaz, amely számos lépést, és így több rekordot kellene hozhatók létre az állapottároló – mindegyiket egy egyéni lépést állapotát leíró.

A Feladatütemező is a feldolgozói szerepkör részeként fut, és megvalósítja az üzleti logika, amely kezeli a sorrendben. Az ütemező új rendelések lekérdezési példányának megvizsgálja az állapottároló rekordok ahol a `LockedBy` mező értéke null, és a `ProcessState` mező függőben. Amikor az ütemező megtalál egy új sorrendben, akkor azonnal feltölti a `LockedBy` mezőt a saját Példányazonosító beállítása a `CompleteBy` mező egy megfelelő időpontot, és beállítja a `ProcessState` mezőjét a feldolgozásra. A kód tervezték kizárólagos és atomi, ha biztosítani szeretné, hogy a Feladatütemező két egyidejű példánya nem egyidejűleg kezelni ugyanabban a sorrendben.

Az ütemező futtat a rendelés feldolgozása aszinkron módon történik, az üzleti munkafolyamat átadja azt az értéket a `OrderID` mezőjét az állapottárolóhoz. A munkafolyamat sorrendjének kezelése a rendelés részleteit lekéri a rendelések adatbázist, és a tevékenységeket hajt végre. Amikor egy lépéseként rendelés feldolgozása kell meghívni a távoli szolgáltatás, az ügynök. A munkafolyamat-lépés a ügynök két kérelem/válasz csatorna működő Azure Service Bus-üzenetsorok használatával kommunikál. Az ábra mutatja a megoldás magas szintű nézetét.

![2. ábra – a Feladatütemező ügynök felügyelő mintát használ az Azure-megoldásban rendelések kezeléséhez](./_images/scheduler-agent-supervisor-solution.png)

A munkafolyamat-lépés a az ügynök küldött üzenet ismertetése, és végezze el által időpontot. Ha az ügynök választ kap a távoli szolgáltatás, a befejezéséhez által idő lejárta előtt, a Service Bus-üzenetsorba, amelyen a munkafolyamat figyeli a visszaküldés válaszüzenetet küld. A munkafolyamat-lépés a érvényes válasz üzenetet kap, amikor befejeződik a feldolgozás és a Feladatütemező beállítása a "ProcessState mező a sorrend szerinti feldolgozásra. Ezen a ponton a rendelés feldolgozása sikeresen befejeződött.

A befejezéséhez által idő lejárta előtt az ügynök választ kap a távoli szolgáltatás az ügynök egyszerűen leáll a feldolgozás, és leállítja a kezelési sorrendje. Hasonlóképpen ha kezelése sorrendje a munkafolyamat befejezéséhez által idő meghaladja, azt is megszakítja. Mindkét esetben feldolgozását, de a befejezéséhez által idő marad készletében állapot tárolási sorrendjének állapotának azt jelzi, hogy a rendelés feldolgozásra idő telt tekinteni, a folyamat sikertelen volt. Vegye figyelembe, hogy az ügynököt, hozzáférhet a távoli szolgáltatás, vagy a munkafolyamat, milyen sorrendben (vagy mindkettőt) kezelő váratlanul leáll, ha az állapottároló található információk újra marad állítsa be a feldolgozásra, és végül egy lejárt befejezéséhez-által értéke lesz.

Ha az ügynök egy helyreállíthatatlan, nontransient hibát észlel, azt a távoli szolgáltatással való kapcsolatfelvételre tett kísérlet közben, egy hiba választ a munkafolyamat is küldheti. Az ütemező sorrendje állapotának beállítása hibás, és indítson eseményt, amely riasztást küld a kezelőnek. Az operátor is próbálja manuálisan megoldani a hiba okát, és küldje el újra a sikertelen feldolgozási lépést.

A felügyelő rendszeres időközönként megvizsgálja a állapot rendelések lejárt befejezéséhez-által értéket keresi. Ha a felügyelő talál egy olyan rekordot, növeli a `FailureCount` mező. Ha a hiba száma érték a megadott küszöbérték alatti, a felügyelő alaphelyzetbe állítja a `LockedBy` mező null, frissítések a `CompleteBy` mezőt egy új lejárati időt, és beállítja a a `ProcessState` mezőről függőben lévő. Az ütemező példányának itt megadott sorrendben átvételéhez, és hajtsa végre, a feldolgozás előtt. Ha a hiba száma értéke meghaladja a megadott küszöbértéket, a hiba okát nontransient adottnak. A felügyelő hiba a rendelés állapotát állítja be, és kivált egy eseményt, amely riasztást küld a kezelőnek.

> Ebben a példában a felügyelő vezettek be egy külön feldolgozói szerepkör. Stratégiák számos segítségével gondoskodik a felügyelő feladat futtatásának, beleértve a Azure Scheduler szolgáltatás használatakor (amelyek nem tévesztendők az ebben a mintában az ütemező összetevő). Az Azure Scheduler szolgáltatás kapcsolatos további információkért látogasson el a [Feladatütemező](https://azure.microsoft.com/services/scheduler/) lap.

Jóllehet nem ebben a példában látható, a Feladatütemező módosítania kell a folyamatát és a sorrend szerez tudomást sorrendje beküldött tartani. Az alkalmazás és az ütemező el különítve egymástól összes függőséget közöttük elkerülése érdekében. Az alkalmazás rendelkezik, amelynek nincsenek saját ismeretei az ütemező példányának kezeli a sorrendben, és az ütemező nem észleli a mely adott alkalmazáspéldány közzétett sorrendje.

Ahhoz, hogy a rendelés állapota jelentendő, az alkalmazás használhatja a saját személyes válaszvárólistáját. A válasz várólista részleteit tartalmazzák a folyamat, amely lenne a következő információk belefoglalása az állapottárolóhoz küldött kérelem a lenne. Az ütemező volna majd Yammerhez a várólistának a rendelés (kérelem érkezett, sorrendben befejeződött, sorrendje nem sikerült és így tovább) állapotát jelzi. Az bele kell foglalni a rendelés azonosítója ezek az üzenetek, azokat is egyeztetés szükséges az eredeti kérést az alkalmazás.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:
- [Ismételje meg a minta][retry-pattern]. Az ügynök ebben a mintában segítségével átlátható módon próbálja megismételni a műveletet fér hozzá a távoli szolgáltatás vagy az erőforrás, amely korábban nem sikerült. Akkor használja, ha az elvárás, hogy a hiba oka átmeneti és javíthatóak.
- [Áramköri megszakító mintát](circuit-breaker.md). Az ügynök ebben a mintában segítségével hibák elhárításához, amikor csatlakozik a távoli szolgáltatás vagy az erőforrás a változó időn tévő kezelni.
- [Kompenzációs tranzakció mintát](compensating-transaction.md). Ha a munkafolyamat egy ütemező által végzett nem fejeződik be sikeresen, korábban végrehajtott munka visszavonása szükség lehet. A tranzakció Compensating mintát ismerteti, hogyan ez érhető el a végleges konzisztencia modell az alábbi műveletek. Az ilyen típusú műveletek általában a egy összetett üzleti folyamatokat és munkafolyamatok végző Feladatütemező valósítják meg.
- [Aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). A Feladatütemező ügynök felügyelő mintában az összetevők általában egymástól leválasztott futtatni, és aszinkron kommunikációhoz. Néhány a módszerekkel, amelyek segítségével valósítja meg az üzenet-várólistákból alapján aszinkron kommunikációt foglalja össze.
- [Vezető választás mintát](leader-election.md). A műveleteket, megakadályozhatja, hogy a megkísérli a folytatást sikertelen ugyanezt a felügyelő több példány koordinálására szükség lehet. A vezető választás mintát ehhez ismerteti.
- [Architektúra: A Feladatütemező-ügynök – felügyelő mintát](https://blogs.msdn.microsoft.com/clemensv/2010/09/27/cloud-architecture-the-scheduler-agent-supervisor-pattern/) Clemens Vasters blog
- [Folyamat Manager minta](http://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)
- [6 – referencia: Egy Saga a Sagas](https://msdn.microsoft.com/library/jj591569.aspx). Egy példa látható, hogyan CQRS mintát használja-e a folyamat manager (a CQRS út útmutatást része).
- [A Microsoft Azure Scheduler](https://azure.microsoft.com/services/scheduler/)

[retry-pattern]: ./retry.md
