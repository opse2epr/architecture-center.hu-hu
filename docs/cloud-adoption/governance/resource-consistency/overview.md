---
title: 'CAF: Erőforrás konzisztencia fegyelem áttekintése'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás konzisztencia fegyelem áttekintése
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ce29efab1502d59513528ab4b640173346aee516
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899190"
---
# <a name="caf-resource-consistency-discipline-overview"></a>CAF: Erőforrás konzisztencia fegyelem áttekintése

Erőforrás konzisztencia az egyik a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) belül a [CAF cégirányítási modell](../overview.md). Ez fegyelem módját a működési környezetben, alkalmazás vagy munkaterhelés kezelésével kapcsolatos házirendek összpontosít. IT-üzemeltetési csapatokra gyakran biztosítanak az alkalmazások, számítási feladatok és az eszközintelligencia teljesítményének figyelését. Másik gyakori hajtsa végre a skálázási igények figyelembevételével készült, a szolgáltatói szerződés (SLA) teljesítménytúllépéseknek javítása szükséges feladatokat, és kerülje el proaktív módon SLA teljesítménytúllépéseknek automatikus szervizelés útján. Felhőalapú irányítási öt állapítják belül erőforrás konzisztencia egy szakterületi biztosíthatja, hogy az erőforrások konzisztens módon oly módon, hogy azok felfedhető informatikai műveletek szerepelnek a helyreállítási megoldások és lehet előre telepített be vannak konfigurálva megismételhető működési folyamatok.

> [!NOTE]
> Konzisztencia erőforrás-szabályozás nem helyettesíti a meglévő informatikai csapatok, folyamatok és eljárások, amelyek lehetővé teszik a szervezet számára, hogy hatékonyan felügyelni a felhőbeli erőforrásokat. Ezen a területen elsődleges célja, hogy a lehetséges üzleti kockázatok azonosítása és kockázatcsökkentő útmutatást nyújt az informatikai munkatársak, amely a felhőbeli erőforrások kezeléséért. Ki, a cégirányítási szabályzatokat és folyamatok ügyeljen arra, hogy megfelelő IT-csoportoknak bevonja a tervezési és folyamatok ellenőrzése.

Ez a szakasza a CAF ismerteti, hogyan hozhat létre egy erőforrás konzisztencia fegyelem a cégirányítási stratégia részeként. Az elsődleges Ez az útmutató célközönségét a szervezet felhőben dolgozó tervezők és más tagjai a Felhőbeli Cégirányítási csapat. Azonban a döntéseket hozhat, a házirendek és a folyamatokat, amelyek a fegyelem vet fel üzemeltetésicsoport- és a hozzászólások megfelelő tagjaival, telepítését és felügyeletét a szervezet erőforrás konzisztencia-megoldások feladata az IT-csoportoknak.

Ha a szervezet nem rendelkezik az erőforrás konzisztencia stratégiák szakértelem, fontolja meg a vonzó külső tanácsadók a fegyelem részeként. Megfontolnia a vonzó [Microsoft Consulting Services programmal](https://www.microsoft.com/enterprise/services), a [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) bevezetési szolgáltatás, vagy más külső felhő bevezetését szakértők felhőalapú rendszerezheti, nyomon követheti, hogyan lehet a legjobban megvizsgálja és a felhőalapú erőforrások használatának legoptimálisabb.

## <a name="policy-statements"></a>Házirend-utasítások

Gyakorlatban hasznosítható házirend-utasítások és az eredményül kapott architektúra követelmények kiszolgálása egy erőforrás konzisztencia fegyelem alapjaként. A házirend utasítás minták, olvassa el a cikk a [erőforrás konzisztencia házirend-utasítások](./policy-statements.md). Ezek a minták a szervezet cégirányítási szabályzatok kiindulási pontként szolgálhat.

> [!CAUTION]
> A mintául szolgáló házirendek közös felhasználói felületéről származnak. Jobban ezek a házirendek adott felhőbeli cégirányítási igényeihez igazítása, hajtsa végre az alábbi lépéseket, amelyek megfelelnek az egyedi üzleti házirend-utasítások létrehozásához szüksége van.

## <a name="developing-resource-consistency-governance-policy-statements"></a>Fejlesztés erőforrás konzisztencia cégirányítási házirend-utasítások

Az alábbi hat lépést a példák és megfontolandó szempontokra erőforrás konzisztencia cégirányítási lehetséges módszert kínálnak. Minden egyes lépést használva kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, és az érintett üzleti és informatikai csapata hozza létre a szervezetben szabályzatoknak és folyamatoknak erőforrás konzisztencia kockázatok enyhítéséhez szükséges létesíteni.

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
                        <h3>Erőforrás konzisztencia sablon</h3>
                        <p class="x-hidden-focus">Egy erőforrás konzisztencia fegyelem dokumentálja a sablon letöltése</p>
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
                        <p class="x-hidden-focus">Ismerje meg a okokra és az erőforrás konzisztencia fegyelem informatikához kapcsolódó kockázatokat.</p>
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
                        <p class="x-hidden-focus">A mutatók megértéséhez, hogy az erőforrás konzisztencia fegyelem beruházni a megfelelő időben.</p>
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
                        <p class="x-hidden-focus">A szabályzatoknak való megfelelés támogatása az erőforrás konzisztencia fegyelem javasolt folyamatokat.</p>
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
                        <p class="x-hidden-focus">Azure-szolgáltatások, amely támogatja az erőforrás konzisztencia fegyelem implementálhatók.</p>
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
