---
title: 'CAF: Erőforrás konzisztencia szabályzat megfelelőségi folyamat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás konzisztencia szabályzat megfelelőségi folyamat
author: BrianBlanchard
ms.openlocfilehash: 26c2d60635a98a3ce061352979ddf01426947e08
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899263"
---
# <a name="resource-consistency-policy-compliance-processes"></a>Erőforrás konzisztencia szabályzat megfelelőségi folyamat

Ez a cikk ismerteti a szabályzat betartása folyamatokat, amelyek szabályozzák egy megközelítést [erőforrás konzisztencia](./overview.md). Hatékony felhőalapú erőforrás konzisztencia cégirányítási működési elégtelenségekkel azonosítása, üzembe helyezett erőforrások kezelését, és győződjön meg arról, üzleti szempontból alapvető fontosságú számítási feladatait van minimális fennakadások tervezett ismétlődő manuális folyamatokat kezdődik. Ezeket a manuális folyamatokat kiegészítik az automation figyeléssel és eszközök segítségével csökkentheti a cégirányítási gyűjteményértékelést kelljen végezni, és lehetővé teszik a szabályzat eltérés gyorsabban választ.

## <a name="planning-review-and-reporting-processes"></a>Tervezési, tekintse át és jelentéskészítési folyamatainak

Felhőalapú rendszerek felügyeleti eszközök tömbjét adja meg, és rendszerezheti, kiépítését, amelyekkel funkciók méretezhetők, és minimalizálják az állásidőt. Jól ezek az eszközök használatával hatékonyan struktúra, és a magánfelhők számára, hogy a potenciális kockázatok csökkentése üzemeltetéséhez szükséges úgy Gondoltuk, folyamatok és üzleti résztvevőkkel és informatikai üzemeltetési csapatok szoros együttműködés mellett házirendeket.

Az alábbiakban látható példa folyamatok gyakran erőforrás konzisztencia tantárgy készletét. Ezekben a példákban használja kiindulási pontként, a folyamatokat, amely lehetővé teszi, hogy frissítse üzleti változás- és a fejlesztési visszajelzései alapján erőforrás konzisztencia-szabályzat, és biztosítja az IT-csoportoknak műveletekké útmutatást bekapcsolásával tervezésekor.

**A hitelkockázat értékelése és tervezése a kezdeti**: A kezdeti bevezetését az erőforrás konzisztencia fegyelem részeként azonosítsa az alapvető üzleti kockázatok és üzemeltetési és informatikai felügyeleti kapcsolódó tolerancia. Ezt az információt használja adott technikai kockázatok egyeztetnie, tagok számítási feladatok tulajdonosai számára tervezett, kockázatok csökkentése érdekében erőforrás konzisztencia házirendek alapkészletével fejlesztése és informatikai csapata hozza létre a kezdeti adatirányítási stratégia kialakítása.

**Üzembe helyezés megtervezése**: Bármely eszköz telepítés előtt minden olyan új működési kockázatok azonosításához felülvizsgálat végrehajtása. Erőforrás-követelmények vonatkoznak, és várt igény szerinti létrehozása, illetve méretezhetőséget igénylő és esetleges használati optimalizálási lehetőségek azonosításához. Emellett ellenőrizze, biztonsági mentési és helyreállítási tervek vannak érvényben.

**Központi telepítés tesztelése**: Központi telepítésének részeként a felhő Cégirányítási munkatársaival, a felhő üzemeltetői csapatokat együttműködve lesz felelős erőforrás konzisztencia szabályzatoknak való megfelelés ellenőrzése az üzembe helyezés áttekintése.

**Éves tervezési**: Az évente hajtsa végre az erőforrás konzisztencia stratégia magas szintű áttekintése. Fedezze fel a vállalati későbbi bővítésre csomag vagy a prioritások, és frissítse a felhő bevezetésének stratégiák kockázati számának potenciális növekedése vagy egyéb felmerülő erőforrás konzisztencia igények azonosítására. Ezúttal segítségével-felhő erőforrás konzisztencia a legújabb ajánlott eljárásokat és ezek a szabályzatok és folyamatok ellenőrzése.

**Negyedéves áttekintése és tervezési**: Negyedévente hajtsa végre az operatív adatokat, és azokat a változásokat az erőforrás konzisztencia-szabályzat szükséges az incidensekkel kapcsolatos jelentéseket. Ez a folyamat részeként tekintse át a módosításokat az erőforrások használatát és teljesítményét növekedések és csökkenések az erőforrás-elosztás igényelnek, és bármilyen számítási feladatot vagy a deduplikációra használatból való kivonást egyaránt az eszközök azonosítására eszközök azonosításához.

A tervezési folyamat akkor is felfedeznie a felhő Cégirányítási csapata új és változó házirend és a egy szakterületi, erőforrás konzisztencia kapcsolatos kapcsolatos Tudásbázis hézagok aktuális tagságától kiértékeléséhez. A felülvizsgálat és ideiglenes technikai tanácsadók vagy a csoport állandó tagjai tervezési megfelelő informatikai munkatársak meghívása.

**Oktatást és képzést**: Bi-havi alapon tanfolyamokat, hogy informatikai szakemberek és fejlesztők naprakész-e a legújabb erőforrás konzisztencia házirend követelményeinek, és útmutatást nyújtanak. Ez a folyamat részeként át, és frissítheti a valamilyen írásos dokumentációt vagy más képzési eszközöket annak biztosítására, hogy a legújabb vállalati szabályzat utasításokat szinkronban.

**Havi naplózási és jelentéskészítési felülvizsgálatok**: Havi rendszerességgel történik hajtsa végre a naplózást ahhoz, hogy biztosítsa azok folyamatos ciklustól erőforrás konzisztencia-szabályzat minden felhőalapú üzemelő példányok esetében. Tekintse át az informatikai részleg kapcsolódó tevékenységeket, és azonosítani a megfelelőségi problémákat már nem kezeli a folyamatban lévő figyelési és érvényesítési folyamat részeként. A felülvizsgálat eredménye az Felhőstratégia és minden felhő bevezetését való kommunikációhoz az általános teljesítmény és a szabályzat betartása a jelentést. A jelentés is tárolja a naplózási és jogi célú.

## <a name="ongoing-monitoring-processes"></a>Folyamatban lévő figyelési folyamatokat

Amely meghatározza, hogy ha az erőforrás konzisztencia adatirányítási stratégia sikeres függ bepillantást nyerhetnek a felhőalapú infrastruktúra jelenlegi és korábbi állapotát. Anélkül, hogy a lényeges metrikák és a felhőalapú környezet állapotát és tevékenységét az adatok elemzése lehetővé teszi nem módosítások azonosítható a kockázatokat, és a kockázati tolerancia megsértésének észlelése. A folyamatban lévő cégirányítási folyamatokat a fent ismertetett szükséges minőség házirend módosíthatók az felhőalapú erőforrások felhasználásának optimalizálása és a felhőben futó számítási feladatok teljesítményének növelése érdekében.

Győződjön meg arról, hogy az IT-csoportoknak valósította automatikus monitorozási rendszerek a felhőalapú infrastruktúra, amely a megfelelő naplók adatok kell értékelnie a kockázatokat. Proaktív szemlélet érvényesítését a kérdés észlelési és kockázatenyhítő lehetséges szabályzat megsértése, hogy ezek a rendszerek figyelése, és győződjön meg, hogy a monitorozási stratégia az operatív igényeinek megfelelően.

## <a name="violation-triggers-and-enforcement-actions"></a>Megsértése triggereket és műveleteket végrehajtó

Erőforrás konzisztencia szabályzatoknak való megfelelés kritikus szolgáltatáskimaradás vagy jelentős költséget meghaladása kockázatok vezethet, mivel a felhő Cégirányítási csapat meg nem felelés incidensek betekintést kell rendelkeznie. Győződjön meg arról, informatikai munkatársak rendelkezik egyértelmű eszkalációs elérési utakat a jelentéskészítés ezeket a problémákat a legjobb cégirányítási csapat tagjainak azonosításához, és győződjön meg arról, hogy a házirend problémák vannak üzemeltetésbiztonságának alkalmas.  

Szabálysértés észlelése esetén, a műveletek minél hamarabb újraigazítás szabályzattal kell tennie. Az informatikai csapat a legtöbb megsértése eseményindítók a leírt eszközök segítségével automatizálhatja a [erőforrás konzisztencia eszközláncban az Azure-ban](toolchain.md).

Következő eseményindítók és műveletek kényszerítési adja meg mikor hivatkozhat példák megoldásához szabálymegsértéseknek monitorozási adatok használatáról:

- Észlelt a szükségesnél több erőforrással ellátott erőforrás: Erőforrások észlelt használatával kevesebb mint 60 %-a Processzor- vagy memóriakapacitása automatikusan horizontális vagy megszüntetés erőforrásokat, csökkentheti a költségeket.
- Észlelt underprovisioned erőforrás: Erőforrások használatával észlelt több mint 80 %-a CPU és memória-kapacitás kell automatikus vertikális felskálázás vagy további kapacitást biztosít további erőforrások kiépítése.
- Címkézetlen erőforrás-létrehozás: Minden szükséges metaadatokat címkék nélküli erőforrás létrehozására vonatkozó kérelem automatikusan a rendszer elutasítja.
- Kritikus fontosságú erőforrás kimaradás észlelhető: Informatikai személyzetet tart fenn a kritikus fontosságú leállások észlelt leállásaihoz értesítést kap. Ha nem azonnal feloldható szolgáltatáskimaradás, a munkatársak, továbbítja a problémát, és számítási feladatok és a felhő Cégirányítási csapat értesítése. A felhő Cégirányítási csapat az jövőbeli incidensek megelőzését szükség a szabályzat változat a probléma nyomon felbontást és frissítési útmutató-ig.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-folyamatok és eseményindítók, amelyek a jelenlegi cloud bevezetési tervet.

A felhő-felügyeleti szabályzatok végrehajtása bevezetési terveket ciklustól útmutatásért lásd: a cikk a fegyelem fokozása.

> [!div class="nextstepaction"]
> [Erőforrás konzisztencia fegyelem fokozása](./discipline-improvement.md)
