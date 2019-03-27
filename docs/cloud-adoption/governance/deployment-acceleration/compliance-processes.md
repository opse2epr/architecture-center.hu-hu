---
title: 'CAF: Gyorsulás szabályzat megfelelőségi folyamatok'
description: Gyorsulás szabályzat megfelelőségi folyamatok
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: alexbuckgit
ms.openlocfilehash: d469c3ba6fb53c5412c615886d449d9b59c2378e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299294"
---
# <a name="deployment-acceleration-policy-compliance-processes"></a>Gyorsulás szabályzat megfelelőségi folyamatok

Ez a cikk ismerteti a szabályzat betartása folyamatokat, amelyek szabályozzák egy megközelítést [üzembe helyezési gyorsítás](./overview.md). Cloud-konfiguráció hatályos irányítása az ismétlődő manuális folyamatokat, a hibák észlelése, és ezek a kockázatok csökkentése érdekében házirendek kivetett kezdődik. Azonban ezeket a folyamatokat automatizálhatja és kiegészítik az eszközök csökkentheti a cégirányítási gyűjteményértékelést kelljen végezni, és lehetővé teszik a szórás gyorsabban választ.

## <a name="planning-review-and-reporting-processes"></a>Tervezési, tekintse át és jelentéskészítési folyamatainak

A felhőben a legjobb telepítési gyorsítás eszközök csak annyira jók a folyamatok és a házirendeket, amelyek támogatják-e. Az alábbiakban látható egy biztonsági alaptervet fegyelem részeként gyakran használt példa folyamatok készletét. Ezekben a példákban használja kiindulási pontként, üzleti változás- és a biztonsági és IT-csoportoknak irányításra vonatkozó műveletekké bekapcsolásával felelős visszajelzései alapján a folyamatokat, amely lehetővé teszi, hogy a biztonsági szabályzat frissítésének megtervezése.

**A hitelkockázat értékelése és tervezése a kezdeti**: A kezdeti bevezetését az üzembe helyezés gyorsítás fegyelem részeként azonosítsa az alapvető üzleti kockázatot és a kapcsolódó üzleti alkalmazásai üzembe helyezését az eltérések. Ezt az információt használja adott technikai kockázatok egyeztetnie, az IT-üzemeltetői csapatnak, és a biztonsági csapat tagjai, valamint hozhat létre a központi telepítési és konfigurációs szabályzatok létrehozására, a kezdeti adatirányítási stratégia kockázatok csökkentése alapkészletével.

**Üzembe helyezés megtervezése**: Bármely eszköz üzembe helyezése előtt minden olyan új kockázatok azonosítása, és minden hozzáférés és az adatok biztonsági házirend követelményeinek teljesítése biztonsági felülvizsgálat végrehajtása.

**Központi telepítés tesztelése**: Bármely eszköz a telepítési folyamat részeként a vállalati biztonsági csoportokkal együttműködésben felhőalapú Cégirányítási csapat felelős biztonsági szabályzatoknak való megfelelés ellenőrzése az üzembe helyezés áttekintése.

**Éves tervezési**: Éves magas szintű áttekintése telepítési gyorsítás stratégia végez. Fedezze fel a jövőbeni vállalati prioritások és frissített cloud bevezetési stratégia lehetséges kockázati növekedését és más új konfigurációt igénylő és lehetőségek azonosításához. Ezúttal segítségével a legutóbbi üzembe helyezés gyorsítás ajánlott eljárásokat és ezek a szabályzatok és folyamatok ellenőrzése.

**Negyedéves áttekintése és tervezési**: Negyedéves felülvizsgálja biztonsági naplózási adatok és a szükséges telepítési gyorsítás szabályzatban azokat a változásokat az incidensekkel kapcsolatos jelentéseket. Ez a folyamat részeként a tekintse át a jelenlegi kiberbiztonsági világának proaktív módon szivattyúi az újonnan felbukkanó fenyegetésekkel szemben, és szükség szerint szabályzat frissítése. A felülvizsgálat befejezése után igazítása az alkalmazások és rendszerek tervezési útmutatást a frissített szabályzatot.

A tervezési folyamat akkor is felfedeznie a felhő Cégirányítási csapata új és változó házirend és a DevOps és az üzembe helyezés gyorsítás kapcsolatos kapcsolatos Tudásbázis hézagok aktuális tagságától kiértékeléséhez. A felülvizsgálat és ideiglenes technikai tanácsadók vagy a csoport állandó tagjai tervezési megfelelő informatikai munkatársak meghívása.

**Oktatást és képzést**: Bi-havi alapon ellenőrizze, hogy informatikai szakemberek és fejlesztők naprakész-e a legújabb gyorsítás központi telepítési stratégiát, és a követelmények tanfolyamokat kínálnak. Ez a folyamat részeként át, és frissítheti a valamilyen írásos dokumentációt, útmutatást vagy egyéb képzési eszközök szinkronban vannak a legújabb vállalati szabályzat utasításokat biztosításához.

**Havi naplózási és jelentéskészítési felülvizsgálatok**: Hajtsa végre a havi naplózást ahhoz, hogy biztosítsa a konfigurációs házirend folyamatos igazításukhoz központi telepítések cloud. Tekintse át az informatikai részleg a biztonsággal kapcsolatos tevékenységeket, és azonosítani a megfelelőségi problémákat már nem kezeli a folyamatban lévő figyelési és érvényesítési folyamat részeként. A felülvizsgálat eredménye az Felhőstratégia és minden felhő bevezetését való kommunikációhoz házirend általános betartása a jelentést. A jelentés is tárolja a naplózási és jogi célú.

## <a name="ongoing-monitoring-processes"></a>Folyamatban lévő figyelési folyamatokat

Amely meghatározza, hogy ha a központi telepítési gyorsítás adatirányítási stratégia sikeres függ bepillantást nyerhetnek a felhőalapú infrastruktúra jelenlegi és korábbi állapotát. Anélkül, hogy a lényeges metrikák és a felhőalapú erőforrások biztonsági állapotát és tevékenységét az adatok elemzése lehetővé teszi nem módosítások azonosítható a kockázatokat, és a kockázati tolerancia megsértésének észlelése. A fent ismertetett folyamatos irányítás folyamatok minőség adatok házirend is módosítani kell az infrastruktúra helytelenül konfigurált erőforrások történő módosítása a fenyegetések és a kockázatok elleni védelme biztosításához szükséges.

Győződjön meg arról, hogy a biztonsági és informatikai csapata hozza létre automatizált monitorozási rendszerek a felhőalapú infrastruktúra, amely a megfelelő naplók adatok kell értékelnie a kockázati végrehajtották. Proaktív szemlélet érvényesítését a kérdés észlelési és kockázatenyhítő lehetséges szabályzat megsértése, hogy ezek a rendszerek figyelése, és győződjön meg arról, megfelelően telepítési és konfigurációs a monitorozási stratégia igényeinek megfelelően.

## <a name="violation-triggers-and-enforcement-actions"></a>Megsértése triggereket és műveleteket végrehajtó

Nem felel meg a konfigurációs szabályzatok kritikus szolgáltatás megszakadása kockázatok vezethet, mivel a felhő Cégirányítási csapat súlyos szabálymegsértéseknek betekintést kell rendelkeznie. Győződjön meg arról, informatikai munkatársak rendelkezik egyértelmű eszkalációs elérési utakat a jelentéskészítés konfigurációs megfelelőségi problémák a legjobb cégirányítási csapat tagjainak azonosításához, és győződjön meg arról, hogy a házirend problémák vannak üzemeltetésbiztonságának alkalmas.  

Szabálysértés észlelése esetén, a műveletek minél hamarabb újraigazítás szabályzattal kell tennie. Az informatikai csapat a legtöbb megsértése eseményindítók a leírt eszközök segítségével automatizálhatja a [üzembe helyezési gyorsítás eszközláncban az Azure-ban](toolchain.md).

Következő eseményindítók és műveletek kényszerítési példákkal használhatja, ha megvizsgálja a monitorozási adatok használata a szabálymegsértéseknek megoldásához:

- **Konfigurációs észlelt a váratlan módosítások**. Ha egy erőforrás konfigurációjának váratlanul módosul, együttműködve informatikai munkatársak és a számítási feladatok tulajdonosai okát, és a egy javítási tervet fejlesztése.
- **Új erőforrások nem követi a szabályzatot.** Megismerheti a fejlesztési és üzemeltetési csapatok és számítási feladatok tulajdonosai számára a központi telepítési gyorsítás házirendek áttekintése projekt indítása során, így mindenki érintett felismeri, hogy a megfelelő házirend követelményeinek.
- **Üzembe helyezési hibák vagy a konfigurációs problémák késésekhez vezethet, a projekt ütemezéseket.** Fejlesztői csapatok és a számítási feladatok tulajdonosai számára, hogy a csapat tisztában van azzal, hogyan automatizálható a felhőalapú erőforrások a konzisztencia és ismételhetőség érdekében végzett munka. Teljes mértékben automatizált központi telepítések meg kell követelni a fejlesztési ciklus korai szakaszában &mdash; szeretne elvégezni ezt a fejlesztési ciklus késői általában vezet kapcsolatos váratlan problémák és késések jelentkezhetnek.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-folyamatok és eseményindítók, amelyek a jelenlegi cloud bevezetési tervet.

A felhő-felügyeleti szabályzatok végrehajtása bevezetési terveket ciklustól útmutatásért lásd: a cikk a fegyelem fokozása.

> [!div class="nextstepaction"]
> [Üzembe helyezés gyorsítás fegyelem fokozása](./discipline-improvement.md)
