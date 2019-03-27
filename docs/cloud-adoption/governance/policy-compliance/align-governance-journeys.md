---
title: 'CAF: Tervezési útmutatók szabályzattal igazítása'
description: Hogyan igazítása tervezési útmutatók házirenddel?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298438"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a>Hogyan igazítása tervezési útmutatók házirenddel?

Miután [meghatározott felhőalapú házirendek](define-policy.md) alapján a [azonosított kockázatok](understanding-business-risk.md), kell létrehozni, amely igazodik az informatikai részleg és a fejlesztők számára, hogy tekintse meg ezeket a házirendeket a hasznos útmutatást. Tervezése felhőalapú cégirányítási tervezési útmutató lehetővé teszi, hogy adja meg adott szerkezeti, technológiai, és a folyamat lehetőségek az öt cégirányítási állapítják minden egyes létrehozott szabályzat utasítások alapján.

Egy felhőalapú cégirányítási kialakítási útmutató kell létesíteni az architektúrát és tervezési minták minden, felhőben üzemelő példánya, amely a legjobban megfelel a házirend követelményeinek infrastruktúrájának alapvető összetevői. Ezek mellett adja meg a technológia, eszközöket és folyamatokat, amelyek támogatni fogja az alábbi tervezési döntéseket minden egyes magas szintű leírását.

Bár a kockázati elemzés és a házirend utasítások előfordulhat, hogy, hogy bizonyos fokon cloud platform-agnosztikus, lehet a kialakítási útmutató kell biztosítania platformspecifikus megvalósítási részletek, amelyek az informatikai RÉSZLEG és dev. Tervezési döntés meghozatalakor az architektúra, eszközök és a választott platformtól funkciók összpontosíthat, és útmutatást nyújtunk.

Felhőbeli tervezési útmutatók figyelembe kell vennie néhány az infrastruktúra-összetevőkhöz kapcsolódó technikai részletek, amíg nem jogosultak széles körű műszaki dokumentumok vagy specifikációk kell lennie. Ellenőrizze, hogy az összes házirend-utasítások és egyértelműen állapot tervezési döntéseket könnyen formátumú személyzetnek ismertetése és oldja meg az útmutatók.

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a>A gyakorlatban hasznosítható cégirányítási Journey használatával

Ha az Azure platform használatához a felhő bevezetésére vonatkozóan, a CAF biztosít [cégirányítási Journey](../journeys/overview.md) ábrázoló a növekményes megközelítés a CAF cégirányítási modell. Ezek az nyújt narratíva utak általános bevezetési forgatókönyveket, például az üzleti kockázatok, követelményeket a tolerancia és adtuk ki egy termék létrehozásához keres cégirányítási minimális működőképes (MVP) házirend-utasítások számos terjed ki. Ezek az utak képviseli egy összefoglaló, való életből vett vásárlói élmény, a felhő bevezetésének folyamatát az Azure-ban.

Míg minden felhőre való áttérés egyedi célok prioritások és kihívásokat, ezek a minták kell adja meg egy megfelelő sablont a házirend átalakítása útmutatást. Válassza ki a legközelebbi forgatókönyvet az adott helyzethez kiindulási pontként, és elképzelt adott csoportházirend igényeihez igazítva.

## <a name="next-steps"></a>További lépések

Tervezési útmutatást helyezze el, és szabályzat betartása folyamatok szabályzatoknak való megfelelőség biztosításához kialakítása.

> [!div class="nextstepaction"]
> [Szabályzat betartása folyamatok](processes.md)