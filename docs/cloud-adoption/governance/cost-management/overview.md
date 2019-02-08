---
title: 'CAF: A Cost Management fegyelem'
description: Felhőalapú cégirányítási viszonyítva Cost Management ismertetése
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: a86b1a75da57cec9c9aaf88abb70049f70731561
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899223"
---
# <a name="cost-management-discipline-overview"></a>A Cost Management fegyelem áttekintése

A Cost Management az egyik a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) belül a [CAF Cégirányítási modell](../overview.md). Sok ügyfél, a költségek szabályozása esetén a fő szempont a felhőalapú technológiák bevezetése. Terheléselosztás kapacitásigényű pacing elfogadása és a cloud services költségek kihívást jelenthet. Ez a különösen fontos, amelyek alkalmazzák a felhőalapú technológiák jelentős üzleti átalakítások során. Ez a szakasz ismerteti a megközelítés a Cost Management fegyelem tapasztalatlan a felhő adatirányítási stratégia részeként.  

> [!NOTE]
> A Cost Management cégirányítási nem helyettesíti a meglévő üzleti csapatok, számlázási eljárások és eljárások, amelyek szerepet játszanak a szervezet pénzügyi informatikai kapcsolatos költségek kezelését. Ezen a területen elsődleges célja, hogy azonosítsa a felhővel kapcsolatos kockázatokat kapcsolatos esetleges informatikai költségeit, és kockázatcsökkentő útmutatást nyújt az üzleti és informatikai csapata hozza létre felelős, üzembe helyezése és kezelése a magánfelhők számára. Ki, cégirányítási szabályzatokat és a folyamatok ellenőrizze, hogy a megfelelő üzleti és informatikai munkatársak bevonja a tervezési és folyamatok ellenőrzése.

Az elsődleges Ez az útmutató célközönségét a szervezet felhőben dolgozó tervezők és más tagjai a Felhőbeli Cégirányítási csapat. Azonban a döntéseket hozhat, a házirendek és a folyamatokat, amelyek a fegyelem vet fel üzemeltetésicsoport engagement és az üzleti és informatikai csapata hozza létre, különösen ezeket a tulajdonos, kezeléséhez és kellene fizetnie felelős vezető tagjai megfelelő folytatott felhőalapú számítási feladatokhoz.

## <a name="policy-statements"></a>Házirend-utasítások

Gyakorlatban hasznosítható házirend-utasítások és az eredményül kapott architektúra követelmények szolgálja ki a Cost Management fegyelem alapjaként. A házirend utasítás minták, olvassa el a cikk a [Cost Management házirend-utasítások](./policy-statements.md). Ezek a minták a szervezet cégirányítási szabályzatok kiindulási pontként szolgálhat.

> [!CAUTION]
> A mintául szolgáló házirendek közös felhasználói felületéről származnak. Jobban ezek a házirendek adott felhőbeli cégirányítási igényeihez igazítása, hajtsa végre az alábbi lépéseket, amelyek megfelelnek az egyedi üzleti házirend-utasítások létrehozásához szüksége van.

## <a name="developing-cost-management-governance-policy-statements"></a>Fejlesztés a Cost Management cégirányítási házirend-utasítások

Az alábbi hat lépést segít a költségek csökkentését a környezetben a cégirányítási szabályzatokat határozhat meg.

<!-- markdownlint-disable MD033 -->

<ul  class="panelContent cardsE">
<li style="display: flex; flex-direction: column;">
    <a href="./template.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-template.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>A Cost Management-sablonnal</h3>
                        <p class="x-hidden-focus">A Cost Management fegyelem dokumentálja a sablon letöltése</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li><li style="display: flex; flex-direction: column;">
    <a href="./business-risks.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-risks.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Üzleti kockázat</h3>
                        <p class="x-hidden-focus">A Cost Management fegyelem informatikához kapcsolódó kockázatokat és okokra ismertetése.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./metrics-tolerance.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-metrics.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Mutatók és metrikák</h3>
                        <p class="x-hidden-focus">Megismerheti, hogy a megfelelő időben a Cost Management fegyelem támogatásán mutatók.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./compliance-processes.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-enforce.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Szabályzat betartása folyamatok</h3>
                        <p class="x-hidden-focus">A szabályzatoknak való megfelelés támogatása a Cost Management fegyelem javasolt folyamatokat.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./discipline-improvement.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-maturity.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Lejárat</h3>
                        <p class="x-hidden-focus">A felhő bevezetésének fázisai a Felhőfelügyelet lejárat igazítása.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./toolchain.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-toolchain.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Eszközlánc</h3>
                        <p class="x-hidden-focus">Azure-szolgáltatások, amely támogatja a Cost Management fegyelem implementálhatók.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>További lépések

Első lépésként egy adott környezetben üzleti kockázatot kiértékelésekor.

> [!div class="nextstepaction"]
> [Üzleti kockázatai ismertetése](./business-risks.md)

<!-- markdownlint-enable MD033 -->
