---
title: 'CAF: Üzembe helyezés gyorsítás fegyelem – áttekintés'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Üzembe helyezési gyorsítás magyarázat felhőalapú cégirányítási viszonyítva.
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 8a9cd359f631708d07ab601c4488b969dc8294a0
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299291"
---
# <a name="caf-deployment-acceleration-discipline-overview"></a>CAF: Üzembe helyezés gyorsítás fegyelem – áttekintés

Üzembe helyezés gyorsítás az egyik a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) belül a [CAF Cégirányítási modell](../overview.md). Ez fegyelem módját a házirendeket szabályozó eszköz konfigurációs vagy telepítési összpontosít. Felhőalapú Cégirányítási öt állapítják üzembe helyezési gyorsítás telepítési, konfigurációs igazítási és parancsfájl újrahasznosíthatóság magában foglalja. Ez lehet manuális tevékenységek vagy teljesen automatizált fejlesztési és üzemeltetési tevékenységeket. Mindkét esetben a házirendek lenne nagyjából ugyanazok maradnak. Ez fegyelem kiforrottá, a felhő Cégirányítási csapat alapul szolgálhat egy partnert a DevOps és az üzembe helyezés stratégiák felgyorsítása a központi telepítések és akadályainak felhőre, az újrafelhasználható eszközök az alkalmazáson keresztül.

Ez a cikk, amely egy vállalati tervezése, létrehozása, bevezetése és működtetése egy felhőalapú megoldás végrehajtási fázis során találkozik gyorsítás üzembe helyezési folyamatának ismertetése. Nem lehet bármely, az összes olyan cégek igényeit kielégítő fiók egy dokumentum. Ez a cikk szakaszonként emiatt a javasolt minimális és a potenciális tevékenységek vázolja fel. Ezek a tevékenységek célja segíteni egy [házirend MVP](../policy-compliance/overview.md#minimum-viable-product-mvp-for-policy), de keretrendszer megállapítása [növekményes házirend](../policy-compliance/overview.md#incremental-policy-growth) fejlődést szem előtt tartva. A felhő Cégirányítási csapat kell döntenie, hogy mennyit fektethet be ezeket a tevékenységeket a központi telepítési gyorsítás pozíció javítása érdekében.

> [!NOTE]
> A központi telepítési gyorsítás fegyelem nem helyettesíti a meglévő informatikai csapatok, folyamatok és eljárások, amelyek lehetővé teszik a szervezet számára hatékonyan telepítheti és konfigurálhatja a felhőbeli erőforrásokat. Ezen a területen elsődleges célja, hogy a lehetséges üzleti kockázatok azonosítása és kockázatcsökkentő útmutatást nyújt az informatikai munkatársak, amely a felhőbeli erőforrások kezeléséért. Ki, a cégirányítási szabályzatokat és folyamatok ügyeljen arra, hogy megfelelő IT-csoportoknak bevonja a tervezési és folyamatok ellenőrzése.

Az elsődleges Ez az útmutató célközönségét a szervezet felhőben dolgozó tervezők és más tagjai a Felhőbeli Cégirányítási csapat. Azonban a döntéseket hozhat, a házirendek és a folyamatokat, amelyek a fegyelem vet fel üzemeltetésicsoport és a hozzászólások megfelelő tagjaival, az üzleti és informatikai csapata hozza létre, különösen azokat központi telepítését és konfigurálását a felhő alapú felelős vezető számítási feladatokhoz.

## <a name="policy-statements"></a>Házirend-utasítások

Gyakorlatban hasznosítható házirend-utasítások és az eredményül kapott architektúra követelmények kiszolgálása egy központi telepítési gyorsítás fegyelem alapjaként. A házirend utasítás minták, olvassa el a cikk a [üzembe helyezési gyorsítás házirend-utasítások](./policy-statements.md). Ezek a minták a szervezet cégirányítási szabályzatok kiindulási pontként szolgálhat.

> [!CAUTION]
> A mintául szolgáló házirendek közös felhasználói felületéről származnak. Jobban ezek a házirendek adott felhőbeli cégirányítási igényeihez igazítása, hajtsa végre az alábbi lépéseket, amelyek megfelelnek az egyedi üzleti házirend-utasítások létrehozásához szüksége van.

## <a name="developing-deployment-acceleration-governance-policy-statements"></a>Fejlesztés a központi telepítési gyorsítás cégirányítási házirend-utasítások

Az alábbi hat lépést segítséget nyújt a cégirányítási szabályzatok meghatározása a vezérlő üzembe helyezését és konfigurálását az erőforrások a felhőalapú környezetben.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsE">
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
                        <h3>A központi telepítési sablont-gyorsítás</h3>
                        <p class="x-hidden-focus">A központi telepítési gyorsítás fegyelem dokumentálja a sablon letöltése</p>
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
                        <p class="x-hidden-focus">Ismerje meg a okokra és a központi telepítési gyorsítás fegyelem informatikához kapcsolódó kockázatokat.</p>
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
                        <p class="x-hidden-focus">Megismerheti, hogy a megfelelő időben a központi telepítési gyorsítás fegyelem támogatásán mutatók.</p>
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
                        <p class="x-hidden-focus">A szabályzatoknak való megfelelés támogatása a központi telepítési gyorsítás fegyelem javasolt folyamatokat.</p>
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
                        <p class="x-hidden-focus">Azure-szolgáltatások, amely támogatja a központi telepítési gyorsítás fegyelem implementálhatók.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>További lépések

Első lépésként kiértékelése [üzleti kockázatai](./business-risks.md) egy adott környezetben.

> [!div class="nextstepaction"]
> [Üzleti kockázatai ismertetése](./business-risks.md)

<!-- markdownlint-enable MD033 -->