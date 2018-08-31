---
title: CQRS architektúrastílus
description: A CQRS-architektúrák előnyeit, illetve az azzal kapcsolatos problémákat és az ajánlott eljárásokat ismerteti
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: ba7af25f940a01e184279c4665f8fce8ebb71b23
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325924"
---
# <a name="cqrs-architecture-style"></a>CQRS architektúrastílus

A Command and Query Responsibility Segregation (CQRS, azaz a parancskiadási és a lekérdezési felelősségek elkülönítése) egy olyan architektúrastílus, amely elkülöníti az olvasási műveleteket az írási műveletektől. 

![](./images/cqrs-logical.svg)

A hagyományos architektúrák esetében ugyanaz az adatmodell használatos az adatbázisok lekérdezésére és frissítésére. Ez egyszerű és jól működik, ha alapszintű CRUD-műveletekről van szó. Összetettebb alkalmazások esetében azonban ez a megközelítés nehézkessé válhat. Például az olvasási oldalon az alkalmazás számos különböző lekérdezést végezhet, amelyek különböző típusú adatátviteli objektumokat (DTO-kat) adnak vissza. Az objektumok leképezése igen bonyolulttá válthat. A írási oldalon a modell összetett érvényesítési és üzleti logikát valósíthat meg. Ennek eredményeképpen egy túlzottan összetett, túl sok feladatot végző modell jöhet létre.

Egy másik lehetséges probléma, hogy az olvasási és írási számítási feladatok gyakran aszimmetrikusak, amelyek teljesítménybeli és méretezhetőségi igényei nagyban eltérhetnek egymástól. 

A CQRS ezeket a problémákat úgy oldja meg, hogy az olvasást és az írást önálló modellekbe különíti: az adatok frissítéséhez **parancsokat**, az olvasásukhoz pedig **lekérdezéseket** használ.

- A parancsoknak feladatalapúnak kell lenniük ahelyett, hogy adatközpontúak lennének. (Például „Book hotel room” a „set ReservationStatus to Reserved” helyett.) A parancsok a szinkron feldolgozás helyett egy aszinkron feldolgozási sorba kerülnek.

- A lekérdezések soha nem módosítják az adatbázist. A lekérdezés egy olyan DTO-t ad vissza, amely nem tartalmaz a környezettel kapcsolatos ismerteket.

A nagyobb mértékű elkülönítés érdekében fizikailag is elkülönítheti az olvasási és az írási adatokat. Ebben az esetben az olvasási adatbázis használhatja a saját, lekérdezésekhez optimalizált adatsémáját. Például tárolhatja az adatok [materializált nézetét][materialized-view], a bonyolult illesztések vagy O/RM-leképezések elkerülése érdekében. Sőt, akár eltérő típusú adattárat is használhat. Például az írási adatbázis lehet relációs, míg az olvasási adatbázis lehet egy dokumentum-adatbázis.

Különálló olvasási és írási adatbázisok használata esetén az adatbázisokat szinkronban kell tartani. Ez általában úgy érhető el, hogy az írási modell az adatbázis frissítésekor közzé tesz egy eseményt. Az adatbázis frissítésének és az esemény közzétételének egy tranzakción belül kell megtörténnie. 

Egyes CQRS-megvalósítások az [Event Sourcing mintát][event-sourcing] használják. Ebben a mintában az alkalmazásállapot események sorozataként tárolódik. Az egyes események az adatok módosításainak egy halmazát jelölik. A jelenlegi állapot az események visszajátszása alapján áll össze. Egy CQRS-környezetben az Event Sourcing egyik előnye, hogy ugyanazok az események használhatók a többi összetevő, jelen esetben az olvasási modell értesítéséhez. Az olvasási modell az események használatával pillanatképet készít az aktuális állapotról, ami a lekérdezések esetében hatékonyabb megoldásnak bizonyul. Az Event Sourcing azonban bonyolultabbá teszi a kialakítást.

![](./images/cqrs-events.svg)

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

A CQRST olyan együttműködési tartományok esetében érdemes használni, ahol több felhasználó dolgozik ugyanazokkal az adatokkal, különösen, ha az olvasási és írási számítási feladatok aszimmetrikusak.

A CQRS nem legfelső szintű architektúra, amely a teljes rendszerre vonatkozik. A CQRST-t csak azokra az alrendszerekre alkalmazza, ahol az olvasás és az írás különválasztása egyértelműen megéri. Különben csak nagyobb fokú összetettséget hoz létre, különösebb előny nélkül.

## <a name="benefits"></a>Előnyök

- **Függetlenül méretezhető**. A CQRS lehetővé teszi az olvasási és írási számítási feladatok egymástól független méretezését, ezáltal kevesebb zárolási versenyt eredményezhet.
- **Optimalizált adatsémák.**  Az olvasási oldal lekérdezésekre optimalizált sémát, az írási oldal pedig frissítésekhez optimalizált sémát használhat.  
- **Biztonság**. Egyszerűbb meggyőződni arról, hogy csak a megfelelő tartományi entitások végeznek írást az adatokon.
- **Kockázatok elkülönítése**. Az olvasási és írási oldalak elkülönítésével könnyebben fenntartható és rugalmas modellek hozhatók létre. A bonyolult üzleti logika legnagyobb része az írási modellbe kerül. Az olvasási modell lehet viszonylag egyszerű.
- **Egyszerűbb lekérdezések**. A materializált nézet olvasási adatbázisban való tárolásával elkerülhető, hogy az alkalmazásnak bonyolult illesztésekre legyen szüksége a lekérdezések során.

## <a name="challenges"></a>Problémák

- **Összetettség**. A CQRS alapvető működése egyszerű. Viszont bonyolultabbá teheti az alkalmazások kialakítását, különösen akkor, ha az Event Sourcing mintát is tartalmazza.

- **Üzenetkezelés**. Bár a CQRS használatához nincs szükség üzenetkezelésre, az üzenetkezelési szolgáltatást gyakorta használják a parancsok feldolgozására és a frissítési események közzétételére. Ebben az esetben az alkalmazásnak kezelnie kell az üzenethibákat és az ismétlődő üzeneteket. 

- **Végleges konzisztencia**. Ha elkülöníti az olvasási és írási adatbázisokat, az olvasási adatok elavulttá válhatnak. 

## <a name="best-practices"></a>Ajánlott eljárások

- A CQRS használatával kapcsolatos további információkért lásd: [CQRS minta][cqrs-pattern].

- A frissítések ütközésének elkerülése érdekében fontolja meg az [Event Sourcing][event-sourcing] minta használatát.

- Az olvasási modell esetében fontolja meg a [Materialized View minta][materialized-view] használatát a séma a lekérdezésekhez való optimalizálása érdekében.

## <a name="cqrs-in-microservices"></a>CQRS használata a mikroszolgáltatásokban

A CQRS különösen hasznos lehet [mikroszolgáltatásokra épülő architektúrák esetében][microservices]. A mikroszolgáltatások egyik alapelve, hogy a szolgáltatások nem érhetik el közvetlenül egy másik szolgáltatás adattárát.

![](./images/cqrs-microservices-wrong.png)

Az alábbi diagramon az A szolgáltatás egy adattárba ír, a B szolgáltatás pedig megőrzi az adatok materializált nézetét. Az A szolgáltatás közzétesz egy eseményt, valahányszor az adattárba ír. A B szolgáltatás feliratkozik az eseményre.

![](./images/cqrs-microservices-right.png)


<!-- links -->

[cqrs-pattern]: ../../patterns/cqrs.md
[event-sourcing]: ../../patterns/event-sourcing.md
[materialized-view]: ../../patterns/materialized-view.md
[microservices]: ./microservices.md
