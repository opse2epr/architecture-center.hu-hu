---
title: 'CAF: Biztonsági alapkonfiguráció szabályzat megfelelőségi folyamat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapkonfiguráció szabályzat megfelelőségi folyamat
author: BrianBlanchard
ms.openlocfilehash: 8f115436d7021fe725118dcc0dfd64167c4cbfa1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899378"
---
# <a name="security-baseline-policy-compliance-processes"></a>Biztonsági alapkonfiguráció szabályzat megfelelőségi folyamat

Ez a cikk ismerteti a szabályzat betartása folyamatokat, amelyek szabályozzák egy megközelítést [biztonsági alapterv](./overview.md). A cloud security hatékony irányítása biztonsági rések és írnak elő a biztonsági kockázatok csökkentésének lehetőségeit házirendek az ismétlődő manuális folyamatokat kezdődik. Ehhez normál bevonása felhőalapú Cégirányítási csapat és az érdekelt üzleti és informatikai érintettek, tekintse át és a szabályzat frissítése és a szabályzatoknak való megfelelés biztosítása érdekében. Ezenkívül számos folyamatos figyelése és érvényesítési folyamat lehet automatikus vagy csökkentheti a cégirányítási gyűjteményértékelést kelljen végezni, és lehetővé teszik a szabályzat eltérés gyorsabban választ eszközkészletébe egészül ki.

## <a name="planning-review-and-reporting-processes"></a>Tervezési, tekintse át és jelentéskészítési folyamatainak

A felhőben a legjobb biztonsági alapterv eszközök csak annyira jók a folyamatok és a házirendeket, amelyek támogatják-e. Az alábbiakban látható példa folyamatok gyakran a biztonsági alapkonfiguráció fegyelem készletét. Ezekben a példákban használja kiindulási pontként, üzleti változás- és a biztonsági és informatikai csapata hozza létre, irányításra vonatkozó műveletekké bekapcsolásával biztosítja visszajelzései alapján a folyamatokat, amely lehetővé teszi, hogy a biztonsági szabályzat frissítésének megtervezése.

**A hitelkockázat értékelése és tervezése a kezdeti**: A kezdeti bevezetését az alapvető biztonsági fegyelem részeként azonosítsa az alapvető üzleti kockázatot és az adott felhő biztonsággal kapcsolatos. Ezt az információt használja adott technikai kockázatok egyeztetnie, az informatikai RÉSZLEG tagjai és biztonsági csapatok és a biztonsági szabályzatok létrehozására, a kezdeti adatirányítási stratégia kockázatok csökkentése alapkészletével fejlesztése.

**Üzembe helyezés megtervezése**: Bármilyen számítási feladatok vagy az eszköz telepítése előtt minden olyan új kockázatok azonosítása, és minden hozzáférés és az adatok biztonsági házirend követelményeinek teljesítése biztonsági felülvizsgálat végrehajtása.

**Központi telepítés tesztelése**: Bármilyen számítási feladatot, a felhő Cégirányítási csapat vagy eszköz számára a telepítési folyamat részeként a vállalati biztonsági együttműködve teamsben felelős biztonsági szabályzatoknak való megfelelés ellenőrzése az üzembe helyezés áttekintése.

**Éves tervezési**: Az évente hajtsa végre az alapvető biztonsági stratégia magas szintű áttekintése. Fedezze fel a jövőbeni vállalati prioritások és frissített cloud bevezetési stratégia, kockázati számának potenciális növekedése és egyéb felmerülő biztonsági igényeinek. Ezúttal segítségével a legújabb biztonsági alapterv ajánlott eljárásokat és ezek a szabályzatok és folyamatok ellenőrzése.

**Negyedéves áttekintése és tervezési**: Negyedévente hajtsa végre a biztonsági naplózási adatokat, és azonosíthatja a biztonsági házirendet a szükséges módosításokat az incidensekkel kapcsolatos jelentéseket. Ez a folyamat részeként a tekintse át a jelenlegi kiberbiztonsági világának proaktív módon szivattyúi az újonnan felbukkanó fenyegetésekkel szemben, és szükség szerint szabályzat frissítése. A felülvizsgálat befejezése után igazítása tervezési útmutatást a frissített szabályzatot.

A tervezési folyamat akkor is felfedeznie a felhő Cégirányítási csapata új és változó házirend és a kapcsolódó biztonsági kockázatok kapcsolatos Tudásbázis hézagok aktuális tagságától kiértékeléséhez. A felülvizsgálat és ideiglenes technikai tanácsadók vagy a csoport állandó tagjai tervezési megfelelő informatikai munkatársak meghívása.

**Oktatást és képzést**: Bi-havi alapon olyan tanfolyamokat, hogy informatikai szakemberek és fejlesztők naprakész-e a legújabb biztonsági házirend követelményeinek. Ez a folyamat részeként át, és frissítheti a valamilyen írásos dokumentációt, útmutatást vagy egyéb képzési eszközök szinkronban vannak a legújabb vállalati szabályzat utasításokat biztosításához.

**Havi naplózási és jelentéskészítési felülvizsgálatok**: Havi rendszerességgel történik hajtsa végre a naplózást ahhoz, hogy biztosítsa a biztonsági házirend folyamatos igazításukhoz összes felhőben üzemelő példányok esetében. Tekintse át biztonsági kapcsolatos tevékenységeket az informatikai részleg, és azonosítani a megfelelőségi problémákat, már nem kezeli a folyamatban lévő figyelési és érvényesítési folyamat részeként. A felülvizsgálat eredménye az Felhőstratégia és minden felhő bevezetését való kommunikációhoz házirend általános betartása a jelentést. A jelentés is tárolja a naplózási és jogi célú.

## <a name="ongoing-monitoring-processes"></a>Folyamatban lévő figyelési folyamatokat

Amely meghatározza, hogy ha a biztonsági adatirányítási stratégia sikeres attól függ, bepillantást nyerhetnek a felhőalapú infrastruktúra jelenlegi és korábbi állapotát. Anélkül, hogy a lényeges metrikák és a felhőalapú erőforrások biztonsági állapotát és tevékenységét az adatok elemzése lehetővé teszi nem módosítások azonosítható a kockázatokat, és a kockázati tolerancia megsértésének észlelése. A folyamatban lévő cégirányítási folyamatokat a fent ismertetett szükséges minőségének biztosítása a szabályzat módosíthatók, amelyekkel jobban megvédheti az infrastruktúra módosítása a fenyegetések és biztonsági követelményekkel szemben.

Győződjön meg arról, hogy a biztonsági és informatikai csapata hozza létre automatizált monitorozási rendszerek a felhőalapú infrastruktúra, amely a megfelelő naplók adatok kell értékelnie a kockázati végrehajtották. Proaktív szemlélet érvényesítését a kérdés észlelési és kockázatenyhítő lehetséges szabályzat megsértése, hogy ezek a rendszerek figyelése, és győződjön meg, hogy a monitorozási stratégia biztonsági igényeinek megfelelően.

## <a name="violation-triggers-and-enforcement-actions"></a>Megsértése triggereket és műveleteket végrehajtó

Mivel vezethet, hogy a kritikus fontosságú biztonsági meg nem felelés és adatok kitettség és a szolgáltatás megszűnésének kockázatok, a felhő Cégirányítási csapat betekintéssel kell rendelkezniük súlyos szabálymegsértéseknek. Győződjön meg arról, informatikai munkatársak rendelkezik egyértelmű eszkalációs elérési utak a biztonsági problémák a cégirányítási csapattagok számára ajánlott az olyan azonosításához, és győződjön meg arról, hogy a házirend problémák vannak üzemeltetésbiztonságának jelentéskészítéshez.  

Szabálysértés észlelése esetén, a műveletek minél hamarabb újraigazítás szabályzattal kell tennie. Az informatikai csapat a legtöbb megsértése eseményindítók a leírt eszközök segítségével automatizálhatja a [biztonsági alapterv eszközláncban az Azure-ban](toolchain.md).

Következő eseményindítók és műveletek kényszerítési adja meg mikor hivatkozhat példák megoldásához szabálymegsértéseknek monitorozási adatok használatáról:

- Növelje az észlelt támadásokkal szemben: Ha bármely erőforrás a találgatásos vagy a DDoS-támadások 25 %-os növekedése, egyeztetnie, informatikai biztonsági személyzet és a számítási feladatok kártérítése meghatározásához tulajdonosa. Nyomon követheti a problémát, és útmutatást frissítéséhez, ha a jövőbeli incidensek megelőzését szükség a házirend-változat.
- Rendezetlen adatok észlelte: Bármilyen adatforrást, egy megfelelő adatvédelmi, biztonsági és besorolási fog rendelkezni a külső hozzáférés megtagadva az adatok tulajdonosa, és a megfelelő szintű adatvédelmet a alkalmazni besorolása telepítéséig üzletmenetre gyakorolt hatás nélkül.
- Biztonsági állapottal kapcsolatos probléma észlelhető: Tiltsa le a hozzáférést minden olyan virtuális gépek (VM), amelyek ismert biztonsági szoftver telepítése vagy hozzáférési vagy kártevő biztonsági rések azonosított, amíg megfelelő javításokat. Frissítse a szabályzat útmutatást áthidalhatók az újonnan észlelt fenyegetésekről.
- Hálózati biztonsági rések észlelte: Bármely nem kifejezetten a hálózati hozzáférési szabályzatai által engedélyezett erőforrásokhoz való hozzáférés váltanak ki riasztást, az informatikai biztonsági szakemberek és a kapcsolódó számítási feladatok felelőse. Nyomon követheti a problémát, és frissíti a útmutatást, ha a házirend változat jövőbeli incidensek megoldásához szükséges.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-folyamatok és eseményindítók, amelyek a jelenlegi cloud bevezetési tervet.

A felhő-felügyeleti szabályzatok végrehajtása bevezetési terveket ciklustól útmutatásért lásd: a cikk a fegyelem fokozása.

> [!div class="nextstepaction"]
> [Biztonsági alapkonfiguráció fegyelem fokozása](./discipline-improvement.md)
