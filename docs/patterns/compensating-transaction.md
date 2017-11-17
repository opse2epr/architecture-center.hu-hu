---
title: "Tranzakció Compensating"
description: "Több lépésből áll, amelyek együtt határozza meg, hogy idővel konzisztenssé művelet által végzett munka visszavonása."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: f8337717c4afd6b558f0da8e1ded3a8071340db7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="compensating-transaction-pattern"></a>Tranzakció mintát Compensating

[!INCLUDE [header](../_includes/header.md)]

A lépésekre, és mindezek együtt meghatározzák az idővel konzisztenssé művelet, ha egy vagy több lépést meghibásodik sorozata által végzett munka visszavonása. A végleges konzisztencia modell az alábbi műveletek általában felhőben üzemeltetett alkalmazások, amelyek megvalósítják az összetett üzleti folyamatokat és munkafolyamatok találhatók.

## <a name="context-and-problem"></a>A környezetben, és probléma

Gyakran a felhőben futó alkalmazások módosíthatják az adatokat. Ezeket az adatokat a előfordulhat, hogy különböző földrajzi helyen lévő különböző adatforrások között ossza szét. Versengés elkerülése érdekében, és javíthatja a teljesítményt elosztott környezetben, egy alkalmazás nem próbálja meg az erős tranzakciós konzisztencia biztosításához. Ehelyett az alkalmazás meg kell valósítania végleges konzisztencia. Ebben a modellben egy tipikus munkaidejét művelet külön lépések egy sorozatát áll. A lépések végrehajtása közben a rendszer szerinti átfogó képet inkonzisztens lehet, de a művelet befejeződött, és az összes lépést végre a rendszer kell válnia konzisztens újra.

> A [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx) információkat nyújt azokról az elosztott tranzakciók miért nem méretezhető well, és a végleges konzisztencia modell ezeket az alapelveket.

A végleges konzisztencia modell kihívást, hogyan kezeli a lépést, amely nem sikerült. Ebben az esetben szükség lehet visszavonja az összes a munkát az előző lépésekben a művelet befejeződött. Azonban az adatok nem egyszerűen állítható vissza, mert az alkalmazás más egyidejű példányok módosulhatott azt. Olyan esetekben, ahol az adatok még nem lett módosítva, egyidejű példánya, még akkor is, egy lépés visszavonása nem egyszerűen lehet kell az eredeti állapot-visszaállítást. Különböző üzleti vonatkozó szabályok alkalmazásához szükséges lehet (lásd a példa részben leírt utazás webhely).

Egy művelet, amely megvalósítja a végleges konzisztencia több heterogén adattárolókhoz is, ha visszavonása a művelet lépéseit szükséges minden egyes adattár pedig látogató. Minden-tárolóban végzett munka kell vonható megbízhatóan, hogy a rendszer megakadályozza az fennmaradó inkonzisztens.

Nem minden adat, amely megvalósítja a végleges konzisztencia művelet hatással lehet, hogy tárolható egy adatbázis. Szolgáltatás objektumorientált architektúra (SOA) környezetben egy művelet sikerült el művelet egy szolgáltatás, és a szolgáltatás által tárolt állapota megváltozhat. A művelet visszavonja, az állapotváltozás kell is vonható vissza. Ez a szolgáltatás újra meghívása és egy másik művelet, amely az első eredő megfordítja végrehajtása magába foglaló.

## <a name="solution"></a>Megoldás

A megoldás, hogy a kompenzációs tranzakció alkalmazza. A lépéseket egy kompenzációs tranzakcióban vissza kell vonnia az eredeti művelet lépéseit hatásait. Előfordulhat, hogy a kompenzációs tranzakció nem egyszerűen az állapot, a rendszer nem a elején. a művelet, mert ez a megközelítés végrehajtott változtatások felülírhatják a jelenlegi állapotában más egyidejű alkalmazáspéldányra cserélje. Ehelyett egy intelligens folyamat, amely az egyidejű példányok által végzett munka figyelembe veszi kell lennie. Ez a folyamat általában lesz adott, alkalmazás, az eredeti művelet által végzett munka jellege által vezérelt.

Általános gyakorlatként javasolt, hogy a munkafolyamat használja compensation igénylő idővel konzisztenssé művelet végrehajtásához. Az eredeti műveletet halad, akkor a rendszer minden lépés, és hogyan lehet visszavonni a lépés által végzett munka kapcsolatos adatokat rögzíti. A művelet sikertelen lesz, bármikor, ha a munkafolyamat gyors visszatekerés vissza végig a lépéseken, befejeződött, és a munkát, amely visszavonja az egyes lépéseket hajtja végre. Ne feledje, hogy kompenzációs tranzakció előfordulhat, hogy nem vonható vissza a munkát az eredeti műveletet pontos fordított sorrendben, és lehetséges, párhuzamos egyes visszavonási lépések végrehajtásához.

> Ez a megközelítés hasonlít a tárgyalt Sagas stratégia [Clemens Vasters blog](http://vasters.com/clemensv/2012/09/01/Sagas.aspx).

A kompenzációs tranzakció során is idővel konzisztenssé és is meghiúsulhat. A rendszer kell tudni folytatni a kompenzációs tranzakció helyén sikertelen, és továbbra is. A lépést, amely nem sikerült, ismétlődő kompenzációs tranzakcióban lépéseket definiálni kell az idempotent parancsként, szükség lehet. További információkért lásd: [idempotencia minták](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/) Jonathan Oliver blogjában.

Bizonyos esetekben nem esetleg helyreállítása a lépést, amely manuális beavatkozásra keresztül kivételével nem sikerült. Ilyen helyzetekben a rendszer kell hoz létre riasztást, és adja meg a hiba okát a lehető legtöbb információ.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

Nem lehet könnyen határozható meg, hogy egy lépést, amely megvalósítja a végleges konzisztencia művelet sikertelen volt. A lépés nem meghiúsulhat azonnal, de ehelyett letiltása sikerült. Időtúllépési mechanizmus valamilyen végrehajtásához szükség lehet.

-A compensation logika könnyen nincs általánosítva. Egy kompenzációs tranzakció az adott alkalmazás. Az alkalmazás, elegendő adatnak lehet visszavonni a sikertelen művelettel lévő egyes lépések hatásait támaszkodik.

Az idempotent parancsként kompenzációs tranzakción belül meg kell határozni a lépéseket. Ez lehetővé teszi a lépéseket kell ismételni, ha a kompenzációs tranzakció maga sikertelen.

Az infrastruktúra, amely kezeli a lépéseket az eredeti műveletet, és a kompenzációs tranzakció rugalmas kell lennie. Kell veszítse el egy hibás lépés szükséges információkat, és megbízhatóan nyomon a kompenzáció logika képesnek kell lennie.

A kompenzációs tranzakció nem feltétlenül vissza az adatokat a rendszer, mint az eredeti műveletet elején. Ehelyett azt kiegyenlíti a lépéseket, amelyek sikeresen befejeződött, mielőtt a művelet nem sikerült által végzett munka.

A lépéseket a kompenzációs tranzakcióban sorrendje nem feltétlenül pontos ellenkező az eredeti művelet lépéseit. Például lehet, hogy egy adattár érzékenyebb a inkonzisztenciákat, mint a többi, és így a kompenzációs tranzakció a lépéseket, visszavonja a módosításokat az tárolóba jöjjön létre először.

A rövid távú időkorlát-alapú zárolási helyezi az egyes erőforrások, amely egy művelet befejezéséhez szükséges, és előre lehet beszerezni ezeket az erőforrásokat segítségével érdekében, hogy az általános tevékenység sikeres lesz. A munkahelyi csak azt követően minden erőforrás beszerzett kell végrehajtani. Minden művelet kell, mielőtt kivonatértékeket kér a zárolásokat lejár.

Érdemes lehet újrapróbálkozási logika, amely további forgiving kompenzációs tranzakció kiváltó hibák minimalizálása érdekében a szokásosnál. Ha egy lépés, amely megvalósítja a végleges konzisztencia művelet sikertelen, próbálja meg a hiba átmeneti kivételként kezeli, és ismételje meg a. Csak a leállításához, és kompenzációs tranzakció kezdeményezése, ha a lépés sikertelen, ismételten vagy visszavonhatatlanul.

> Végrehajtási kompenzációs tranzakció kihívásai számos ugyanazok, mint a végleges konzisztencia megvalósítása. Lásd: a végleges konzisztencia végrehajtási szakasz szempontjai a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx) további információt.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a minta csak visszavont kell lennie, ha azt elmulasztják műveletek használják. Ha lehetséges tervezze meg megoldásokat igénylő tranzakciók compensating összetettsége elkerülése érdekében.

## <a name="example"></a>Példa

Utazás webhely lehetővé teszi, hogy az ügyfelek könyv útvonalak. Egyetlen útvonal járatok és szállodák előfordulhat, hogy alkotják. Az ügyfél határozzon London utazik és Párizsi be tudta végezze el az alábbi lépéseket egy útvonal létrehozásakor:

1. Megjelenik egy ülések repülési F1 határozzon London.
2. A felhőszolgáltató közötti átviteléhez F2 London Párizsba helyet könyv.
3. A felhőszolgáltató közötti átviteléhez F3 Párizsi Budapest helyet könyv.
4. Foglaljon le egy helyet, a szállodák H1 Londonban.
5. Egy hely a szállodák H2 megadva lefoglalni.

Ezeket a lépéseket egy idővel konzisztenssé műveletet jelent, bár minden egyes lépést külön műveletként. Ezért is teljesítő ezeket a lépéseket, a rendszer kell is rögzítse szükségessé minden lépés abban az esetben, ha az ügyfél úgy dönt, hogy az útvonal szakítsa meg a számláló műveletek. A számláló műveletek elvégzéséhez szükséges lépések követően futtathatja kompenzációs tranzakcióként.

Figyelje meg, hogy a kompenzációs tranzakció lépéseit nem lehet az eredeti lépéseket pontos ellentéte, és minden lépés kompenzációs tranzakcióban lévő logika figyelembe kell vennie üzleti vonatkozó szabályokat. Például egy repülési hellyel unbooking előfordulhat, hogy nem jogosíthatók az ügyfél a fizetős pénzt teljes visszatérítése. Az ábra azt mutatja be, le egy utazás útvonal hosszú ideig futó tranzakció visszavonása kompenzációs tranzakció létrehozásakor.

![Egy hosszú ideig futó tranzakció le egy utazás útvonal visszavonása kompenzációs tranzakció létrehozása](./_images/compensating-transaction-diagram.png)


> Esetleg a kompenzációs tranzakcióban párhuzamosan, attól függően, hogy az egyes lépéseihez szükséges kompenzációs logikai tervezésétől már végrehajtandó lépéseit.

A számos üzleti megoldások sikertelen volt-e egyetlen lépésben nem mindig szükségessé teszik a működés közbeni a rendszer egy kompenzációs tranzakció használatával ismét. Például ha&mdash;után rendelkező lefoglalt járatok F1 F2 és F3 utazás webhely forgatókönyvben&mdash;az ügyfél nem tudja lefoglalni a helyiségben: Szálloda H1, érdemes biztosítani az ügyfél egy másik Szálloda ugyanabban a városban, egy hely helyett a járatokat megszakítása. Az ügyfél továbbra is dönt, hogy megszakítja a (ebben az esetben az kompenzációs tranzakció fut, és visszavonja a járatok F1 F2 és F3 történt helyfoglalás), de ez a döntés kell tenni, az ügyfél által, nem pedig a rendszer.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). A tranzakció Compensating mintát gyakran használják műveletek visszavonásához, a végleges konzisztencia modell megvalósításához. A kezdés tájékoztatást nyújt az előnyöket és a végleges konzisztencia kompromisszumot.

- [A Feladatütemező-ügynök – felügyelő mintát](scheduler-agent-supervisor.md). Ismerteti, hogyan lehet rugalmas rendszerben elosztott szolgáltatásokat és erőforrásokat használó üzleti műveletek végrehajtásához. Egyes esetekben szükség lehet egy kompenzációs tranzakció használatával egy művelet által végzett munka visszavonása.

- [Ismételje meg a minta](./retry.md). Compensating tranzakciók elvégzéséhez költséges lehet, és lehetséges, egy hatékony házirend tett sikertelen műveletek újrapróbálkozási mintát követve implementálásával minimalizálása érdekében a használatukat.
