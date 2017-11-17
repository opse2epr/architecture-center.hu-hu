---
title: "CQRS architektúra stílus"
description: "Előnyeit, kihívást és ajánlott eljárások a CQRS architektúrák ismerteti"
author: MikeWasson
ms.openlocfilehash: dd3da5886587159f57646ff1bfffa2094725f798
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="cqrs-architecture-style"></a>CQRS architektúra stílus

A parancs és a lekérdezés felelősségi elkülönítése (CQRS) olyan architektúra stílus, amely elválasztja az olvasási műveletek az írási műveletek. 

![](./images/cqrs-logical.svg)

A hagyományos-architektúrák esetén az azonos adatmodell használt lekérdezése, és frissítse az adatbázist. Amely egyszerű és alapvető CRUD műveletek esetén működik. Összetett alkalmazások azonban ez a megközelítés válhat kezelése nehézkessé. Olvasási oldalán, például az alkalmazás végezhet számos különböző lekérdezéseket, különböző alakzatok objektumok (DTOs) adatátvitel vissza. Objektum leképezésének bonyolult válhat. A írási oldalon a modell nehéznek összetett érvényesítési és üzleti logikát. Ennek eredményeképpen fejezheti egy túl összetett modellt, amely túl sok.

Egy másik lehetséges probléma olvasási és írási munkaterhelések gyakran aszimmetrikus, nagyon különböző teljesítmény és méretezhetőség követelményeknek. 

CQRS olvasási elválasztva megoldja ezeket a problémákat, és írja be különálló modellek segítségével **parancsok** frissíteni az adatokat, és **lekérdezések** adatokat olvasni.

- Parancsok kell lennie a feladatot ahelyett, hogy központú adatok alapján. ("Könyv Szálloda helyiségben" nem "beállítás ReservationStatus fenntartott.") Az aszinkron feldolgozás ahelyett, hogy szinkron feldolgozás alatt várólistájának parancsok lehet helyezni.

- Lekérdezések soha ne módosítsa az adatbázis. A lekérdezés egy DTO, amelyek nem foglalják magukban az bármilyen tartomány Tudásbázis adja vissza.

A nagyobb elkülönítési fizikailag elkülönítheti a adatolvasási a az adatok írása. Ebben az esetben olvasható adatbázis használhatja a saját adatok séma optimalizált lekérdezések. Tárolhatja például a [materializált nézet] [ materialized-view] az adatok bonyolult illesztésekre vagy összetett O/RM-hozzárendelések elkerülése érdekében. Adattár más típusú még akkor is használhatja. Például az írási adatbázis esetleg relációs, míg olvasható adatbázis egy dokumentum-adatbázis.

Ha külön olvasási és írási adatbázisokat használnak, azok kell tartani szinkronban. Általában ez érhető el, mivel az egy eseményt, amikor frissíti az adatbázis közzététele írási modell. Az adatbázis frissítése és az esemény közzétételéhez egy tranzakción belül kell megtörténnie. 

Egyes megvalósítások CQRS használati a [esemény forrás mintát][event-sourcing]. Ebben a mintában az alkalmazásállapot események sorozatát tárolja. Minden esemény módosítások készletét reprezentálja, az adatokhoz. A jelenlegi állapota alapján az események visszajátszását összeállított. CQRS környezetben, egy esemény forrásanyag előnye, hogy használható-e az azonos események értesíteni a többi összetevő &mdash; ebben az esetben, értesítse a olvasási modell. Az olvasási modellje a események készít pillanatképet az aktuális állapotát, amely a lekérdezések hatékonyabb. Azonban esemény forrás hozzáadása összetettsége a tervező.

![](./images/cqrs-events.svg)

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

Vegye figyelembe a CQRS sok érhetik el a ugyanazokat az adatokat, különösen akkor, ha az olvasási és írási munkaterhelések aszimmetrikus együttműködési tartományokhoz.

CQRS nincs egy legfelső szintű architektúra, amely egy teljes rendszerre vonatkozik. Csak a alrendszerrel CQRS érvényes Ha olvasási és írási elválasztó egyértelmű értéke van. Ellenkező esetben létrehozásakor nagyobb fokú összetettségével jár nem juttatásra.

## <a name="benefits"></a>Előnyök

- **Egymástól függetlenül skálázás**. CQRS lehetővé teszi, hogy az Olvasás és munkaterhelések méretezését írás, és kevesebb zárolási contentions eredményezheti.
- **Optimalizált adatok sémák.**  Az olvasási oldalán optimalizált séma használható lekérdezéseket, miközben a írási oldalon frissítések optimalizált séma használja.  
- **Biztonság**. Célszerűbb győződjön meg arról, hogy csak a megfelelő tartományhoz entitások hajt végre írási műveleteket az adatokon.
- **Aggályokat elkülönítése**. Az olvasási és írási oldalak elkülönítése modellek, amelyek több fenntarthatóvá és rugalmas eredményezhet. A legtöbb bonyolult üzleti logikát a írási modell állapotba kerül. Lehet, hogy az olvasási modell viszonylag egyszerű.
- **Egyszerűbb lekérdezések**. A materializált nézet tárolása olvasható adatbázis, az alkalmazás elkerülheti a bonyolult illesztésekre lekérdezésekor.

## <a name="challenges"></a>Kihívásai

- **Összetettsége**. CQRS alapvető lényege egyszerű. De azt is vezethet összetettebb alkalmazás tervét, különösen akkor, ha az esemény forrás mintát tartalmaznak.

- **Üzenetküldési**. CQRS nem igényel üzenetküldési, bár általában üzenetküldési folyamat parancsok használata, és (update) események közzététele. Az alkalmazás ebben az esetben üzenet hibák vagy a duplikált üzenetek kell kezelni. 

- **Végleges konzisztencia**. Ha külön az olvasási és írási adatbázisok, előfordulhat, hogy az olvasható adatok elavult. 

## <a name="best-practices"></a>Ajánlott eljárások

- CQRS kapcsolatos további információkért lásd: [CQRS mintát][cqrs-pattern].

- Érdemes lehet a [esemény forrás] [ event-sourcing] mintát frissítés ütközések elkerülése érdekében.

- Érdemes lehet a [materializált nézet mintát] [ materialized-view] olvasási modell a séma lekérdezésekhez optimalizálja.

## <a name="cqrs-in-microservices"></a>A mikroszolgáltatások CQRS

CQRS különösen hasznos lehet egy [mikroszolgáltatások architektúra][microservices]. Mikroszolgáltatások alapelvei egyike, hogy a szolgáltatás nem tud közvetlenül elérni egy másik szolgáltatás adattár.

![](./images/cqrs-microservices-wrong.png)

Az alábbi diagram szemlélteti A szolgáltatás írja az adattárat, és szolgáltatás B tartja az adatok materializált nézet. A szolgáltatás egy eseményt, ha az adattár ír közzéteszi. B szolgáltatás számítógépcsoportra fizetett elő az eseményt.

![](./images/cqrs-microservices-right.png)


<!-- links -->

[cqrs-pattern]: ../../patterns/cqrs.md
[event-sourcing]: ../../patterns/event-sourcing.md
[materialized-view]: ../../patterns/materialized-view.md
[microservices]: ./microservices.md
