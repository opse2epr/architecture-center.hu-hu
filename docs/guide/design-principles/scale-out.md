---
title: Bővíteni. terv
description: A horizontális skálázás úgy kell megtervezni a felhőalapú alkalmazásokhoz.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 8f9b3e99a53f5941f708b0de124f37e6ff7e5ab2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="design-to-scale-out"></a>Bővíteni. terv

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Az alkalmazás, hogy azt horizontálisan méretezhető. terv

Egy elsődleges felhő előnye a rugalmas méretezést &mdash; mértékű kapacitás szükség szerint, a betöltés növekedése kiterjesztése, és ha a plusz kapacitást nincs szükség a méretezés használhatja. Az alkalmazás tervezze meg, hogy azt horizontálisan méretezhető, hozzáadásával vagy eltávolításával új példányok, igény szerint van szüksége.

## <a name="recommendations"></a>Javaslatok

**Példány tölcsérútvonalak elkerülése**. Tölcsérútvonalak, vagy *munkamenet affinitás*, ha ugyanazt az ügyfelet érkező kérelmeket a rendszer mindig irányítja át ugyanazon a kiszolgálón. Tölcsérútvonalak korlátozza az alkalmazás képes kiterjesztése. Például a nagy mennyiségű felhasználó érkező forgalom nem lesz elérhető példányok között. Tölcsérútvonalak okai a munkamenet-állapot tárolása a memóriában, és a számítógép-specifikus kulcsok használatát a titkosításhoz. Győződjön meg arról, hogy bármely példányok kezelik a kérelmet. 

**Szűk keresztmetszetek azonosítása**. Minden teljesítményprobléma magic javítása kiterjesztése nem található. Például a szűk keresztmetszetek háttérbeli adatbázis esetén nem segítséget webkiszolgálókat hozzáadni. Azonosítani és megoldani a szűk keresztmetszetek a rendszer először előtt több példány volt, a problémát. A rendszer állapot-nyilvántartó részei ennek legvalószínűbb oka szűk keresztmetszetek. 

**Munkaterhelések felbontani által méretezhetőségi követelményeinek.**  Alkalmazások gyakran több munkaterhelés méretezéshez eltérő követelmények állnak. Egy alkalmazás Előfordulhat például, egy nyilvánosan elérhető és egy különálló felügyeleti hely. A nyilvános webhely problémákat tapasztalhat a forgalmat, hirtelen emelkedéseit, míg a felügyeleti webhely kisebb, kiszámíthatóbb terhelést. 

**Erőforrás-igényes feladatok kiszervezése.** Nagy mennyiségű Processzor- vagy i/o-erőforrások igénylő feladatokat kell áthelyezni [feladatok háttérben] [ background-jobs] Ha lehetséges, a terhelés, amely kezeli a felhasználói kérelmek kezelőfelületre minimalizálása érdekében.

**Beépített automatikus skálázás használatát**. Sok Azure számítási szolgáltatás kell beépített az automatikus skálázást. Ha az alkalmazás egy előre jelezhető, rendszeres munkaterhelés, horizontális felskálázás ütemezés szerint. Például kibővítési munkaidőben. Ellenkező esetben ha a munkaterhelés nem előre jelezhető, akkor teljesítménymutatók, például a Processzor- vagy kérelem várólistájának hossza való automatikus skálázást. Ajánlott eljárások automatikus skálázást, lásd: [automatikus skálázás][autoscaling].

**Fontolja meg a kritikus munkaterhelésekhez agresszív automatikus skálázás**. A kritikus fontosságú munkaterhelésekhez meg szeretné tartani az igény szerinti előre. Érdemes gyorsan számára a további forgalom kezelésére, és fokozatosan méretezését vissza túl nagy terhelés alatt új példányok felvételéhez.

**Terv a skála**.  Ne feledje, hogy a rugalmasan méretezhető, az alkalmazás időszakok skála, amikor példányok beolvasása törölni kell. Az alkalmazás kell kezelésére eltávolítani kívánt példányokat. Az alábbiakban néhány scalein kezelésének módját:

- Figyelés a leállítási események (ha elérhető) és a leállítási áramtalanították. 
- Ügyfelek, fogyasztók szolgáltatás kell kezelni átmeneti hiba, és próbálkozzon újra. 
- Hosszan futó feladatokat, fontolja meg a munkahelyi, az ellenőrzőpontok használatával összeállításának vagy a [Kiszolgálókészletéhez, és a szűrők] [ pipes-filters-pattern] mintát. 
- Elhelyezése munkaelemek várólistát, hogy egy másik példánya is onnan folytathatja az adatgyűjtést a munkahelyi példány közepén feldolgozási eltávolításakor. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
