---
title: Az üzleti igények összeállítása
description: Minden a tervezési döntés kell igazolni üzleti követelmény
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 110a441ae74334d212a717da2cb038d60b24bb1f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="build-for-the-needs-of-the-business"></a>Az üzleti igények összeállítása

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Minden a tervezési döntés kell igazolni üzleti követelmény

A Tervező elv úgy látszhat, nyilvánvaló, de elengedhetetlen a megoldás tervezésekor vegye figyelembe az. Hajtsa végre, amelyek várhatóan több millió felhasználónak vagy néhány ezer volt? Az egy egy órán keresztül alkalmazás leállás elfogadható? A forgalmat, vagy egy nagyon előre jelezhető munkaterhelés nagy felszakadásáig várható? Végül minden a tervezési döntés igazolnia kell üzleti igény. 

## <a name="recommendations"></a>Javaslatok

**Adja meg az üzleti célkitűzéseiket**, beleértve a helyreállítási idő célkitűzése (RTO), a helyreállítási időkorlát (RPO) és a maximális megengedhető kimaradás (MTO). Ezeket a számokat tájékoztatnia kell a architektúra döntéseket. Például egy alacsony RTO eléréséhez bevezetheti automatikus feladatátvétel egy másodlagos régióban. Azonban ha a megoldás működését egy magasabb RTO, szükségtelen lehet-e a redundancia érdekében, hogy bizonyos fokú.

**Dokumentálja a szolgáltatásszint-szerződések (SLA) és a szolgáltatási szint célkitűzései (SLO)**, beleértve a rendelkezésre állási és teljesítménybeli. Letölti a 99,95 % rendelkezésre állási megoldás lehet, hogy felépítéséhez. Az, hogy elegendő? A válasz-e egy üzleti döntés. 

**Az alkalmazás a vállalati tartomány körül modell**. Indítsa el az üzleti követelmények elemzése. Ezek a követelmények segítségével az alkalmazás a modell. Érdemes lehet egy tartomány-központú kialakítás (nnn) megközelítés létrehozásához [tartomány modellek] [ domain-model] , hogy tükrözze az üzleti folyamatokat és használati esetekben. 

**Rögzítheti funkcionális-és működésképtelenné**. Funkcionális követelményeinek lehetővé teszik, hogy mennyire, hogy az alkalmazás teszi-e a megfelelő művelet. Működik követelmények lehetővé teszik, hogy mennyire, hogy az alkalmazás teszi-e ezeket a beállításokat *is*. Különösen győződjön meg arról, hogy tudomásul veszi a méretezhetőséget, a rendelkezésre állás és a késleltetés követelményeinek. Ezek a követelmények befolyásolják a tervezési döntések és technológia kiválasztása.

**Munkaterhelés szerint felbontani**. Ebben a környezetben "munkaterhelési" kifejezés egy különálló és a számítási feladat, amely logikailag elkülöníthető más feladatok. Előfordulhat, hogy a különböző terhelésekhez rendelkezésre állását, méretezhetőségét, adatkonzisztencia és vész-helyreállítási különböző követelmények vonatkoznak. 

**Tervezze meg a növekedésre**. A megoldás lehet, hogy az igényeinek aktuális, azon felhasználók számát, a kötet tranzakciók, adattárolási és így tovább. Azonban egy robusztus alkalmazás növekedés nagyobb architekturális módosítások nélkül kezeli. Lásd: [horizontális tervezési](scale-out.md) és [körül korlátok partíció](partition.md). Figyelembe venni, hogy az üzleti modelljéhez és üzleti követelmények valószínűleg megváltoznak adott idő alatt. Ha egy alkalmazás szolgáltatási modellt és adatmodellekben túl kemény, válik nehéz fejleszteni az alkalmazást az új használati esetek és forgatókönyvek. Lásd: [alakulása kialakítása](design-for-evolution.md).

**Költségeinek kezelése**. A hagyományos helyszínen alkalmazásban kell fizetnie előzetes megfizetése esetén hardver (CAPEX). A felhő alkalmazásban kell fizetnie a felhasznált erőforrásokat. Győződjön meg arról, hogy tudomásul veszi a szolgáltatások felhasznált árképzési modellt. A teljes költség hálózati sávszélesség, tárolási, IP-címek, szolgáltatások felhasználásához, és más tényezőktől tartalmazza. Lásd: [Azure-beli árakról] [ pricing] további információt. A műveleti költségeket figyelembe venni. A felhőben nem kell kezelni, a hardver- vagy egyéb infrastruktúrával, de továbbra is szeretné kezelni a alkalmazások, beleértve a DevOps, incidensekre adott reakciók, vész-helyreállítási és így tovább. 

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
