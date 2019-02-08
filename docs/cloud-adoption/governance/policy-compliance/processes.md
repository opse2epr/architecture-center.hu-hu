---
title: 'CAF: Ellenőrizni és érvényesíteni a szabályzat betartása'
description: Hogyan gondoskodik az eszközmegfelelőségi szabályzatok a?
author: BrianBlanchard
ms.date: 1/4/2019
ms.openlocfilehash: 9066f33c8baa183476a9632e82d6eb960d03752c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899055"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-processes-can-help-ensure-policy-adherence"></a>Milyen folyamatokat segíthetnek a szabályzat betartása érdekében?

<!---
I've defined policies, I've provided an architecture guide. Now how do I monitor adherence to policy? If there is a violation, how do I enforce the policy?
--->

Létrehozó a felhő házirend-utasítások és a egy kialakítási útmutató tervezése, után kell létrehozni egy stratégiát, hogy a felhőbeli üzembe helyezés marad megfelelnek-e a házirend követelményeinek. Ezt a stratégiát kell foglalnia a felhő Cégirányítási csapata folyamatban lévő felülvizsgálati és kommunikációs folyamatok, mikor szabálymegsértéseknek igényli-e a művelet, és az automatikus figyelési és megfelelőségi rendszerekhez, amelynek az a követelmények meghatározásánál tart feltételek létrehozása megsértésének észlelése és javítási műveletek indításához.

Tekintse meg a vállalati házirend szakasza a [gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md) példákat arra, hogyan illik az szabályzat betartása folyamatot egy felhőbeli cégirányítási terv.

## <a name="prioritize-policy-adherence-processes"></a>Szabályzat betartása folyamatok rangsorolása

Mennyi befektetés folyamatok fejlesztése a házirend céljait támogatásához szükséges? A mérete és a felhőbeli üzemelő példány lejáratú megfelelőségi és az ebből a törekvésből társított költségek támogató folyamatok létrehozásához szükséges munkát nagyban eltérhetnek.

A fejlesztési és tesztelési erőforrásokat tartalmazó kis méretű telepítések a házirend követelményeinek egyszerűek legyenek, és megkövetelik, hogy a cím néhány dedikált erőforrások. A magas prioritású biztonsági és teljesítménybeli igényeinek érett alapvető fontosságú a felhőbeli üzembe helyezés, másrészt a személyzet, széles körű belső folyamatok és egyéni monitorozási eszközök támogatásához a házirend céljait csapat lehet szükség.

A szabályzat betartása stratégia definiálása során az első lépés értékelje ki, a folyamatok alábbiak ismertetik miként támogat a házirend követelményeinek. Határozza meg, mennyi erőfeszítés, amely egyenértékű portfóliónk fejlesztésén ezeket a folyamatokat, és ezen információk használatával valósághű költségvetés és volt szükség létszámnövelésre tervek ezen igények kielégítése érdekében.

## <a name="establish-cloud-governance-team-processes"></a>Felhőalapú Cégirányítási csapat folyamatok létrehozása

Szabályzat megfelelőség szervizelési eseményindítók definiálása, előtt kell kapcsolatot hoz létre, amely a csapat fogja használni a folyamatok és hogyan információt fogja megosztott és informatikai részleg és a felhő Cégirányítási csapat közötti eszkalálva.

### <a name="assign-cloud-governance-team-members"></a>Rendelje hozzá a felhő Cégirányítási csapat tagjai

Kik folyamatban lévő útmutatást nyújtanak a szabályzatoknak való megfelelés és merülnek fel, ha a telepítésével és működtetésével felhőbeli eszköz-házirendekkel kapcsolatos problémák kezelésére? A méretét és a felhő Cégirányítási csapat összetételének a házirend követelményeinek, és a költségvetés és a személyzeti prioritások szabályzatoknak való megfelelés csatolt összetettsége függ.

Válassza ki a csoport tagjai, amelyek szakértelemmel rendelkeznek a megadott házirend-utasítások alá tartozó területeken. Kezdeti tesztkörnyezetek esetében ez lehet néhány felelős rendszergazdák számára a cégirányítási alapjait létrehozó korlátozódik. Az üzemelő példányok részletes, és a szabályzatok összetettebb és a szélesebb körű vállalati házirend követelményeinek szervesebb válik, a felhő Cégirányítási csapat módosítania kell támogatásához, egyre összetettebb házirend követelményeinek.

Az érett, cégirányítási folyamatok rendszeresen annak érdekében, hogy a felhő útmutatást csapat-tagság felülvizsgálata, a legújabb házirend követelményeinek megfelelően kezelheti. Az informatikai részleg tagjai azonosíthatja a megfelelő tapasztalatok jogcímet vagy érdekeltséget cégirányítási speciális területein, és belefoglalhatja őket a teams, szükség szerint állandó vagy ad hoc alapon.

### <a name="reviews-and-policy-iteration"></a>Értékelések és a házirend az iteráció

További források vannak telepítve, mert a felhő Cégirányítási csapat kell győződjön meg arról, hogy új számítási feladatok vagy az eszközök megfelelnek-e a házirend követelményeinek. Tervezze meg a tervezési útmutatók ciklustól vitatni minden új erőforrás telepítéséért felelős csapatok számára.

Az általános üzembe helyezési növekedésével új potenciális kockázatokról rendszeresen értékeli, és szükség szerint frissítse a házirend-utasítások és tervezési útmutatókat. Rendszeres ütemezés felülvizsgálati ciklus egyes az öt cégirányítási oktatnak, hogy a házirend e naprakész, és folyamatban teljesül.

### <a name="education"></a>Oktatás

Szabályzatoknak való megfelelés informatikai részleg és a fejlesztők számára, hogy a házirend követelményeinek, azok a felelősségi területeket érintő megértéséhez szükséges. Tervezze meg az erőforrások dokumentum döntések és követelmények és kapcsolódó csapatok mindegyikével a tervezési útmutatók, amelyek támogatják a házirend követelményeinek a oktatása áldozni.

Az irányelvek változásai esetén, mert rendszeresen frissíteni a dokumentációval és képzéssel anyagok, és oktatási erőfeszítések frissített követelményei és fontos informatikai munkatársak útmutatás kommunikációjának biztosításához.  

### <a name="establish-escalation-paths"></a>Eszkalációs útvonalak létrehozása

Ha egy erőforrás megfelelő, akik lekérdezi az értesítés? Ha az informatikai részleg szabályzat megfelelőségi problémát észlel, aki tegye fel a kapcsolatot? Ellenőrizze, hogy a felhő Cégirányítási csapat eszkalációs folyamata egyértelműen meghatározott. Győződjön meg arról, ezek a kommunikációs csatornák őrzi meg a személyzet és a szervezet változásainak frissített.

## <a name="violation-triggers-and-actions"></a>Megsértése eseményindítók és műveletek

A felhő Cégirányítási csapata és folyamatainak meghatározása, után kell explicit módon határozhatja meg, amelynek az eseményindító műveletek szabálytalanságok mi minősül, és milyen műveletek kell.

### <a name="define-triggers"></a>Eseményindítók definiálása

A házirend-utasítások mindegyike esetében tekintse át a követelményeik alapján hozza meg, mi számít egy szabályzat megsértése. A már létrehozott információk segítségével a szabályzat-definíció folyamat részeként eseményindítók létrehozásához.

* Tolerancia kockázati – a metrikák és a kockázati mutatók, részeként létrehozott alapján megsértése eseményindítók létrehozása a [tolerancia kockázatelemzés](risk-tolerance.md).
* Meghatározott házirend követelményeinek - utasítások rendelkezhetnek a szolgáltatói szerződés (SLA), az üzleti folytonossági és vészhelyreállítási helyreállítási (BRCD) vagy a teljesítményre vonatkozó követelmények, megfelelőségi eseményindítók alapjául használandó házirendet.

### <a name="define-actions"></a>Műveletek definiálása

Minden egyes megsértése trigger rendelkeznie kell a megfelelő műveletet. Aktivált műveletek mindig értesítse-e egy megfelelő informatikai munkatársak vagy a Felhőbeli Cégirányítási csapat egyik tagja a megsértése esetén. Ezt az értesítést a házirendnek való megfeleléssel, vagy egy előre meghatározott szervizelési folyamat függően a kezdeti egy manuális ellenőrzést és az észlelt megsértése súlyossága vezethet.

Néhány példa megsértése eseményindítókat és műveleteket:

| Felhőalapú cégirányítási fegyelem | Minta-eseményindító | Mintául szolgáló művelet |
|-----------------------------|----------------|---------------|
| Cost Management | Havi felhő több mint 20 %-át a vártnál magasabb költségeket. | Értesítse a számlázási vezetője, aki elkezdi az erőforrás-használat áttekintése. |
| Biztonsági alapkonfiguráció | Észleli a gyanús felhasználói bejelentkezési tevékenységet. | Értesíti az IT-biztonsági csoportja, és tiltsa le a gyanús felhasználói fiók. |
| Erőforrás-konzisztencia | CPU-kihasználtság számítási értéke nagyobb, mint 90 %-át. | Értesíti az IT-üzemeltetői csapatnak, és a horizontális felskálázást a terhelés kezeléséhez további erőforrások. |

## <a name="monitoring-and-compliance-automation"></a>Figyelési és megfelelőségi automation

Miután meghatározta a megfelelőségi megsértése triggereket és műveleteket, elkezdheti a legjobb hogyan tervezi használni, a naplózási és jelentéskészítési eszközök és a felhőalapú platform, amely segítségével automatizálhatja a megfigyelés és a szabályzat megfelelőségi stratégia egyéb jellemzőit.

Tekintse át a CAF [naplózás és -jelentéskészítés útmutató döntési](../../decision-guides/log-and-report/overview.md) témakör útmutatást monitorozását végző minta az üzemelő példány a legjobb választás.
