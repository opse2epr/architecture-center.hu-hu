---
title: 'CAF: Felhőszabályozási stratégia implementálása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Ismerje meg a Microsoft Cloud bevezetési keretrendszert Azure (CAF) használatával a felhőbeli adatirányítási stratégia megvalósításához.
author: BrianBlanchard
ms.date: 01/03/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 9da73502ad3a6243572128b5bf72399aad648353
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299388"
---
# <a name="implement-a-cloud-governance-strategy"></a>Felhőszabályozási stratégia implementálása

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Bármilyen üzleti folyamatokat és a technológiai platform változás kockázata üzleti mutatja be. Felhőalapú cégirányítási csapatok, amelynek tagjai is ismertek felhőalapú jegyektől, minimális megszakítással használhatja tovább elfogadását, vagy az innováció eszköztárát kockázatok csökkentése is biztosítja.<br/><br/>Felhőalapú cégirányítási azonban több, mint a gyakorlati megvalósításról igényel. Változás is történt a vállalati nyújt narratíva vagy vállalati házirendek jelentős hatással lehet bevezetési erőfeszítéseket. Végrehajtás előtt fontos, hogy túllépjünk informatikai vállalati szabályzat meghatározása során.<br/><br/>
                </div>
            </div>
        </div>
    </div>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../_images/operational-transformation-govern-highres.png" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize">
            <div class="cardPadding" style="padding-bottom:10px;">
                <div class="card" style="padding-bottom:10px;">
                    <div class="cardText" style="padding-left:0px;">
<img src="../_images/operational-transformation-govern-highres.png" alt="Diagram of the CAF governance model: Corporate policy and governance disciplines">
<br>
<i>1. ábra A vállalati szabályzatok és a felhő Cégirányítási öt oktatnak Vizualizáció</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="define-corporate-policy"></a>Céges szabályzat definiálása

Vállalati házirend meghatározása, azonosítása és üzleti kockázatot, függetlenül a használt felhőplatformtól összpontosít. Kifogástalan állapotú felhőstratégia cégirányítási eredményes vállalati szabályzat kezdődik. A következő három lépésből álló folyamat iteratív fejlesztés házirendeket útmutatók.

<!-- markdownlint-disable MD033 -->

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/understanding-business-risk.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/business-risk.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Üzleti kockázat</h3>
                        <p>Keresse meg a jelenlegi cloud bevezetési terveket és az adatok besorolása üzleti kockázatok azonosításához. Az üzleti egyenleg kockázati hibatűrést és kockázatcsökkentési költségek működnek.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/define-policy.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/corporate-policy.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Szabályzat és megfelelőség</h3>
                        <p>Értékelje ki a szervezet kockázattűrési határát tájékoztatja a minimálisan invazív házirendekben, amelyek szabályozzák a felhőre való áttérés kockázatok csökkentése. Bizonyos szektorokban, a harmadik fél megfelelőségi kezdeti szabályzat létrehozása hatással van.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/processes.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/enforcement.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Folyamatok</h3>
                        <p>A bevezetési és az innováció tevékenységek ütemben természetes módon hoz létre a szabálymegsértéseknek. Kapcsolódó folyamatok végrehajtása fog támogatási figyelésére és az érvényesítési házirendek betartását.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>További lépések

Hatékony felhőalapú adatirányítási stratégia kezdődik üzleti kockázat ismertetése.

> [!div class="nextstepaction"]
> [Üzleti kockázat ismertetése](./policy-compliance/understanding-business-risk.md)
