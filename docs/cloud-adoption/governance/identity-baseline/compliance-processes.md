---
title: 'CAF: Identitás alapkonfiguráció szabályzat megfelelőségi folyamat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás alapkonfiguráció szabályzat megfelelőségi folyamat
author: BrianBlanchard
ms.openlocfilehash: e00427662c811fccb7e0a7261a4bb3dd3437103e
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898753"
---
# <a name="identity-baseline-policy-compliance-processes"></a>Identitás alapkonfiguráció szabályzat megfelelőségi folyamat

Ez a cikk ismerteti a szabályzat betartása folyamatokat, amelyek szabályozzák egy megközelítést [identitásra építve](./overview.md). Hatékony identitás irányítása, hogy az útmutató identitás szabályzat elfogadása és a változatok ismétlődő manuális folyamatokat kezdődik. Ehhez normál bevonása felhőalapú Cégirányítási csapat és az érdekelt üzleti és informatikai érintettek, tekintse át és a szabályzat frissítése és a szabályzatoknak való megfelelés biztosítása érdekében. Ezenkívül számos folyamatos figyelése és érvényesítési folyamat lehet automatikus vagy csökkentheti a cégirányítási gyűjteményértékelést kelljen végezni, és lehetővé teszik a szabályzat eltérés gyorsabban választ eszközkészletébe egészül ki.

## <a name="planning-review-and-reporting-processes"></a>Tervezési, tekintse át és jelentéskészítési folyamatainak

Identitás felügyeleti eszközöket kínálnak a lehetőségeket és szolgáltatásokat, amelyek nagy mértékben segíti a felhasználói és hozzáférés-vezérlési belül üzembe helyezése a felhőben. Azonban akkor is szükség van is úgy Gondoltuk, folyamatok és a szabályzatok hozzáadásával támogatja a szervezet céljai. A következő olyan készlete, például folyamatokat az identitásra építve fegyelem gyakran részt. Ezekben a példákban használja a kiindulási pont, ha üzleti változás, amely lehetővé teszi, hogy az Alkalmazásidentitás házirend frissítése a folyamatok tervezése alapján, és visszajelzést kapjanak az informatikai csapatok irányításra vonatkozó műveletekké bekapcsolásával biztosítja.

**A hitelkockázat értékelése és tervezése a kezdeti**: A kezdeti bevezetését az identitásra építve fegyelem részeként azonosítsa az alapvető üzleti kockázatok és a kapcsolódó felhőalapú identitáskezelést tolerancia. Ezt az információt használja adott technikai kockázatok tárgyalják az identitás-szolgáltatások kezeléséért IT-csapatok olyan tagjai, és fejlessze a biztonsági szabályzatok létrehozására, a kezdeti adatirányítási stratégia kockázatok csökkentése alapkészletével.

**Üzembe helyezés megtervezése**: Minden üzembe helyezés előtt tekintse át a hozzáférés szüksége van a számítási feladatokat, és fejlesszen egy hozzáférés-szabályozási stratégiát, amely igazodik a vállalati identitás szabályzat jön létre. A dokumentum bármely hézagok igényeinek és az aktuális házirend határozza meg, ha a házirend-frissítések szükségesek között, és szükség szerint módosítsa a házirendet.

**Központi telepítés tesztelése**: Az üzembe helyezés részeként a felhő Cégirányítási csapat felelős az identitásszolgáltatások, IT-csoportoknak együttműködve lesz felelős identitás szabályzatoknak való megfelelés ellenőrzése az üzembe helyezés áttekintése.

**Éves tervezési**: Az évente hajtsa végre az identitás-kezelési stratégia magas szintű áttekintése. Ismerje meg a módosításokat, az identity services környezet és a frissített cloud bevezetési stratégiát érintő kockázatok azonosításához növeléséhez, vagy az aktuális identitás-infrastruktúra minták módosítania kell. Ezúttal segítségével tekintse át a legújabb identitáskezelési ajánlott eljárások és ezek a szabályzatok és folyamatok ellenőrzése.

**Negyedévente tervezési**: Negyedévente identitás általános felülvizsgálat végrehajtása és a hozzáférés-vezérlés naplózási adatok, és találkozhat azokkal a felhő bevezetését csapatok bármely esetleges új kockázatokat vagy a működési követelmények, amelyek esetén az Alkalmazásidentitás házirend és a hozzáférés-vezérlés változásai stratégia.

A tervezési folyamat akkor is felfedeznie a kapcsolódó új és változó házirend- és identitáskezelési kapcsolatos Tudásbázis hézagok felhőalapú Cégirányítási csapata aktuális tagságától kiértékeléséhez. A felülvizsgálat és ideiglenes technikai tanácsadók vagy a csoport állandó tagjai tervezési megfelelő informatikai munkatársak meghívása.  

**Oktatást és képzést**: Bi-havi alapon ellenőrizze, hogy informatikai szakemberek és fejlesztők naprakész-e a legújabb szabályzatot identitáskövetelmények tanfolyamokat kínálnak. Ez a folyamat részeként át, és frissítheti a valamilyen írásos dokumentációt, útmutatást vagy egyéb képzési eszközök szinkronban vannak a legújabb vállalati szabályzat utasításokat biztosításához.

**Havi naplózási és jelentéskészítési felülvizsgálatok**: Havi rendszerességgel történik hajtsa végre a naplózást ahhoz, hogy biztosítsa az Alkalmazásidentitás házirend folyamatos igazításukhoz összes felhőben üzemelő példányok esetében. A felülvizsgálat segítségével ellenőrizheti a felhasználói hozzáférés biztosítása a felhasználók megfelelő hozzáférhetnek a felhőbeli erőforrások, és győződjön meg arról, például az RBAC a hozzáférési stratégiákra következetesen teljesültek-módosítás üzleti ellen. Minden olyan kiemelt jogosultságú fiókok határozza meg és dokumentálja a céljukat. A felülvizsgálati folyamat hoz létre egy jelentést, a Felhőstratégia majd minden egyes cloud bevezetési csapat részletesen bemutató átfogó betartásának házirend. A jelentés is tárolja a naplózási és jogi célú.

## <a name="ongoing-monitoring-processes"></a>Folyamatban lévő figyelési folyamatokat

Amely meghatározza, hogy ha az identitás adatirányítási stratégia sikeres attól függ, betekintést az identitáskezelési rendszerek jelenlegi és korábbi állapotát. Elemezheti a felhőbeli üzemelő példány megfelelő metrikák és a kapcsolódó adatok nélkül nem módosítások azonosítható a kockázatok, és a kockázati tolerancia megsértésének észlelése. A folyamatban lévő cégirányítási folyamatokat a fent ismertetett szükséges minőség biztosítása érdekében a szabályzat módosíthatók a változó üzleti igényeinek támogatásához.

Győződjön meg arról, hogy az IT-csoportoknak végrehajtotta-e az identitás-szolgáltatások, amelyek rögzítik a naplókat, és a naplózási információkat kell értékelnie a kockázati automatizált figyelési rendszereket. Proaktív szemlélet érvényesítését a kérdés észlelési és kockázatenyhítő lehetséges szabályzat megsértése, hogy ezek a rendszerek figyelése, és győződjön meg, hogy az Ön identitáskezelési infrastruktúráját módosításokat is megjelennek a monitorozási stratégiában.

## <a name="violation-triggers-and-enforcement-actions"></a>Megsértése triggereket és műveleteket végrehajtó

Az Alkalmazásidentitás házirend megsértésének is bizalmas adatokhoz való illetéktelen hozzáférés eredményez, és az alapvető fontosságú alkalmazások és szolgáltatások súlyos megszakítás vezethet. Szabálysértés észlelése esetén, a műveletek minél hamarabb újraigazítás szabályzattal kell tennie. Az informatikai csapat a legtöbb megsértése eseményindítók a leírt eszközök segítségével automatizálhatja a [identitásra építve eszközlánc](toolchain.md).

Következő eseményindítók és műveletek kényszerítési adja meg mikor hivatkozhat példák megoldásához szabálymegsértéseknek monitorozási adatok használatáról:

- A rendszer gyanús tevékenységet észlelt: Felhasználói bejelentkezést észlelt a névtelen proxy IP-címek, ismeretlen helyről vagy egymást követő bejelentkezések impossibly távoli földrajzi helyekről utalhat egy fiók potenciális támadásokról vagy rosszindulatú hozzáférési kísérlet. Bejelentkezési le lesz tiltva, amíg nem ellenőrizhető a felhasználói identitás és a jelszó alaphelyzetbe állítása.
- Kiszivárogtatott hitelesítő: Felhasználónév és jelszó az internethez kiszivárgott rendelkező fiókok mindaddig, amíg a felhasználó identitása is ellenőrizhető, és új jelszó kérésére le lesz tiltva.
- Észlelt megfelelő hozzáférés-vezérlés: Bármely védett eszközök, ahol korlátozza a hozzáférést nem felelnek meg a biztonsági követelmények lesz hozzáférése blokkolva lesznek, amíg az erőforrás megfelelőségének állapotúra.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-folyamatok és eseményindítók, amelyek a jelenlegi cloud bevezetési tervet.

A felhő-felügyeleti szabályzatok végrehajtása bevezetési terveket ciklustól útmutatásért lásd: a cikk a fegyelem fokozása.

> [!div class="nextstepaction"]
> [Identitás alapkonfiguráció fegyelem fokozása](./discipline-improvement.md)
