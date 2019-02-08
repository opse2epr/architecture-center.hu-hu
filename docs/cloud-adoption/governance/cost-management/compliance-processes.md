---
title: 'CAF: A Cost Management szabályzat megfelelőségi folyamat'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A Cost Management szabályzat megfelelőségi folyamat
author: BrianBlanchard
ms.openlocfilehash: 5fd007546589020f376107382c54c391cc5d05ad
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899058"
---
# <a name="cost-management-policy-compliance-processes"></a>A Cost Management szabályzat megfelelőségi folyamat

Ez a cikk ismerteti egy módszert hozhat létre folyamatokat, amelyek támogatják a [Cost Management](./overview.md) cégirányítási fegyelem. A felhőbeli költségek hatékony cégirányítási szabályzatoknak való megfelelés támogatja az ismétlődő manuális folyamatokat kezdődik. Ehhez a felhő Cégirányítási csapat és a hasznos üzleti résztvevőkkel, tekintse át és a szabályzat frissítése és a szabályzatoknak való megfelelés biztosítása érdekében rendszeres bevonása. Ezenkívül számos folyamatos figyelése és érvényesítési folyamat lehet automatikus vagy csökkentheti a cégirányítási gyűjteményértékelést kelljen végezni, és lehetővé teszik a szabályzat eltérés gyorsabban választ eszközkészletébe egészül ki.

## <a name="planning-review-and-reporting-processes"></a>Tervezési, tekintse át és jelentéskészítési folyamatainak

A felhőben a legjobb Cost Management tools csak annyira jók a folyamatok és a házirendeket, amelyek támogatják-e. Az alábbiakban látható példa folyamatok gyakran részt vesz a Cost Management fegyelem készletét. Ezekben a példákban használja kiindulási pontként, a folyamatokat, amely lehetővé teszi, hogy üzleti változás- és az üzleti csapatok vonatkozik költség-irányításra vonatkozó visszajelzései alapján költség szabályzat frissítésének tervezésekor.

**A hitelkockázat értékelése és tervezése a kezdeti**: A Cost Management tantárgy kezdeti elfogadását részeként azonosítsa az alapvető üzleti kockázatok és a felhőköltségek kapcsolatos tolerancia. Ezen információk használatával bemutatjuk a költségvetés és a költségek kapcsolatos kockázatokat az üzleti csapatok olyan tagjai, és alakítson ki alaptervi házirendek létrehozására, a kezdeti adatirányítási stratégia kockázatok csökkentése.

**Üzembe helyezés megtervezése:** Bármely eszköz telepítené a várt felhőbeli hozzárendelés alapján előre jelzett költségvetési létesíteni. Győződjön meg arról, hogy a központi telepítés tulajdonosi és a naplózási adatok szerződését.  

**Éves tervezési:** Évente hajtsa végre az összes üzembe helyezett és to-be telepített eszköz egy összesítő elemzése. Üzleti egység, részlegek, csapatok és egyéb megfelelő szakaszainak körvonalszínét megjelenő új önkiszolgáló bevezetési költségvetése igazítása Győződjön meg arról, hogy a költségvetés és nyomon követheti a Költekezési annak tudatában, hogy mindegyik számlázási egység vezetője.

Ez az az idő, hogy egy precommitment vagy prepurchase engedmény maximalizálása érdekében. Célszerű év végi kedvezményt lehetőségekről további nagybetűvel a felhő gyártója által biztosított pénzügyi év az éves költségvetés igazítása.

**Negyedévente tervezési:** Negyedévente tekintse át az egyes számlázási vezetője előrejelzést és a tényleges költségek igazítása költségvetése. Ha a szolgáltatáscsomag vagy a nem várt költségkeret minták módosításait, igazítás, és hozzárendeli a költségvetés.

Ez a negyedéves tervezési folyamat akkor is felfedeznie a felhő Cégirányítási csapata Tudásbázis hézagok kapcsolódó jelenlegi vagy jövőbeli üzleti tervek aktuális tagságától kiértékelése. Az érintett személyzet és a számítási feladatok tulajdonosai a felülvizsgálat és ideiglenes tanácsadók vagy a csoport állandó tagjai tervezési meghívása.

**Oktatást és képzést**: Bi-havi alapon ajánlat, hogy az üzleti és informatikai szakemberek naprakészek a Cost Management legújabb házirend követelményeinek a tanfolyamokat. Ez a folyamat részeként át, és frissítheti a valamilyen írásos dokumentációt, útmutatást vagy egyéb képzési eszközök szinkronban vannak a legújabb vállalati szabályzat utasításokat biztosításához.

**Havi jelentés:** Havi rendszerességgel történik a tényleges kiadásokat elleni előrejelzés jelentést. Minden olyan váratlan szorzataként számlázási vezetői értesítést.

Alapszintű folyamatok segítségével igazítása a költségkeret-beállítási, és létrehozza a Cost Management tantárgy alapjait.

## <a name="ongoing-monitoring-processes"></a>Folyamatban lévő figyelési folyamatokat

Egy sikeres adatirányítási stratégia függ, hogy a múltbeli, jelenlegi és tervezett jövőbeni felhővel kapcsolatos betekintést Költségkezelés költségeit. Anélkül, hogy a lényeges mérőszámok és az adatokat a meglévő költségek elemzése lehetővé teszi nem módosítások azonosítható a kockázatokat, és a kockázati tolerancia megsértésének észlelése. A folyamatban lévő cégirányítási folyamatokat a fent ismertetett szükséges minőség annak érdekében, hogy jobban a változó üzleti követelmények alapján infrastruktúra védelmét, és a felhőbeli használati szabályzat módosíthatók.

Győződjön meg arról, hogy az IT-csoportoknak valósította automatikus rendszerek figyeléséhez a felhő költségeit, és nem tervezett gyakorlattól eltérő várható költségeket a használat. Jelentéskészítési és riasztási rendszerek kérdés észlelési és kockázatenyhítő lehetséges szabálymegsértéseknek, hogy hoz létre.

## <a name="compliance-violation-triggers-and-enforcement-actions"></a>Megfelelőségi megsértése eseményindítók és kényszerítési műveletek

Szabálysértés észlelése esetén, kényszerítési házirenddel újraigazítás műveleteket kell tennie. A legtöbb megsértése eseményindítók a leírt eszközök segítségével automatizálhatja a [az Azure Cost Management eszközlánc](toolchain.md).

A következő példák eseményindítók:

* Havi költségvetés szorzataként: Előrejelzés-és-tényleges arány 20 %-kal együtt a számlázási egység vezető keretet túllépő része a havi kiadások eltérések tárgyalják. Rekord felbontásra és előrejelzést változásait.
* Bevezetési ütemezését: Bármilyen eltérés egy előfizetési szinten 20 %-ot meghaladó aktivál egy tekintse át a számlázási vezetője. Rekord felbontásra és előrejelzést változásait.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-folyamatok és eseményindítók, amelyek a jelenlegi cloud bevezetési tervet.

A felhő-felügyeleti szabályzatok végrehajtása bevezetési terveket ciklustól útmutatásért lásd: a cikk a Cost Management fegyelem javítása.

> [!div class="nextstepaction"]
> [A Cost Management fegyelem fokozása](./discipline-improvement.md)
