---
title: 'CAF: Az identitások alapkonfiguráció fegyelem áttekintése'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Magyarázat az identitásra építve felhőalapú cégirányítási viszonyítva
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 14569b8db68434ee30fa7bff7fba2da5fa537049
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298526"
---
# <a name="caf-identity-baseline-discipline-overview"></a>CAF: Az identitások alapkonfiguráció fegyelem áttekintése

Identitásra építve az egyik a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) belül a [CAF Cégirányítási modell](../overview.md). Identitás egyre számít az elsődleges biztonsági határt, a felhőben, amely egy shift a hálózati biztonság hagyományos koncentrálhat. Identitás-szolgáltatások, adja meg a szervezet informatikai környezetekben és hozzáférés-vezérlést támogató core mechanizmusok, és az identitásra építve fegyelem egészíti ki a [biztonsági alapterv fegyelem](../security-baseline/overview.md) következetesen alkalmazásával hitelesítési és engedélyezési követelményeinek cloud bevezetési erőfeszítések között.

> [!NOTE]
> Alapkonfiguráció identitáskezelést nem helyettesíti a meglévő informatikai csapatok, folyamatok és eljárások, amelyek lehetővé teszik a szervezet kezelése és védelme az identitás-szolgáltatások számára. Ezen a területen elsődleges célja, hogy a lehetséges identitáshoz kapcsolódó üzleti kockázatok azonosítása, és adja meg a végrehajtási, karbantartása és az identitás-kezelési infrastruktúra működtetése felelős informatikai munkatársak kockázatcsökkentő útmutatást. Ki, a cégirányítási szabályzatokat és folyamatok ügyeljen arra, hogy megfelelő IT-csoportoknak bevonja a tervezési és folyamatok ellenőrzése.

Ebben a szakaszban a CAF útmutató ismerteti a megközelítés az identitásra építve fegyelem tapasztalatlan a cégirányítási stratégia részeként. Az elsődleges Ez az útmutató célközönségét a szervezet felhőben dolgozó tervezők és más tagjai a Felhőbeli Cégirányítási csapat. Azonban a döntéseket hozhat, a házirendek és a folyamatokat, amelyek a fegyelem vet fel megteszik a kellő engagement, és folytatott fontos tagjai az informatikai csapat felelős megvalósítását és kezelését a szervezet identitáskezelési megoldások.

Ha a szervezet nem rendelkezik az identitásra építve és biztonsági szakértelem, fontolja meg a vonzó külső tanácsadók a fegyelem részeként. Megfontolnia a vonzó [Microsoft Consulting Services programmal](https://www.microsoft.com/enterprise/services), a [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) bevezetési felhőszolgáltatásként vagy más külső felhő bevezetését szakértők vitatni a fegyelem kapcsolatos problémákat.

## <a name="policy-statements"></a>Házirend-utasítások

Gyakorlatban hasznosítható házirend-utasítások és az eredményül kapott architektúra követelmények szolgálja ki, az identitásra építve fegyelem alapjaként. A házirend utasítás minták, olvassa el a cikk a [identitás eredeti házirend-utasítások](./policy-statements.md). Ezek a minták a szervezet cégirányítási szabályzatok kiindulási pontként szolgálhat.

> [!CAUTION]
> A mintául szolgáló házirendek közös felhasználói felületéről származnak. Jobban ezek a házirendek adott felhőbeli cégirányítási igényeihez igazítása, hajtsa végre az alábbi lépéseket, amelyek megfelelnek az egyedi üzleti házirend-utasítások létrehozásához szüksége van.

## <a name="developing-identity-baseline-governance-policy-statements"></a>Fejlesztés az identitásra építve cégirányítási házirend-utasítások

Az alábbi hat lépést a példák és megfontolandó szempontokra identitásra építve cégirányítási lehetséges módszert kínálnak. Minden egyes lépést használva kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, és az érintett üzleti és informatikai csapata hozza létre a szervezetben a szabályzatok és identitással kapcsolatos kockázatok enyhítéséhez szükséges folyamatokat létrehozni.

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
                        <h3>Identitás alapkonfiguráció sablon</h3>
                        <p class="x-hidden-focus">Az identitásra építve fegyelem dokumentálja a sablon letöltése</p>
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
                        <p class="x-hidden-focus">Ismerje meg a okokra és az identitásra építve fegyelem informatikához kapcsolódó kockázatokat.</p>
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
                        <p class="x-hidden-focus">Megismerheti, hogy a megfelelő időben az identitásra építve fegyelem támogatásán mutatók.</p>
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
                        <p class="x-hidden-focus">A szabályzatoknak való megfelelés támogatása az identitásra építve fegyelem javasolt folyamatokat.</p>
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
                        <p class="x-hidden-focus">Azure-szolgáltatások támogatásához az identitásra építve fegyelem implementálhatók.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>További lépések

Első lépésként kiértékelése [üzleti kockázatai](./business-risks.md) egy adott környezetben.

> [!div class="nextstepaction"]
> [Üzleti kockázatai ismertetése](./business-risks.md)
