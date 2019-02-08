---
title: 'CAF: Biztonsági alapkonfiguráció fegyelem áttekintése'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapkonfiguráció fegyelem áttekintése
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 81398ff943a9a582ae3cc29d9ff7519a0d1f8e54
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899186"
---
# <a name="caf-security-baseline-discipline-overview"></a>CAF: Biztonsági alapkonfiguráció fegyelem áttekintése

Alapvető biztonsági az egyik a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) belül a [CAF Cégirányítási modell](../overview.md). Biztonsági bármely informatikai üzembe helyezés részét képező, és a felhő egyedi biztonsági aggályokat vezet be. Számos vállalat vonatkoznak rá szabályozási követelményeknek, amelyek bizalmas adatok védelmének szervezeti nagyobb prioritást egy felhőbeli átalakítás mérlegelése során. A felhőalapú környezetét az esetleges biztonsági fenyegetések azonosítása és a folyamatok és eljárások címzéshez ezek a kártevők kell lennie minden olyan informatikai biztonsági vagy internetes biztonsági csapat számára. A biztonsági alapkonfiguráció fegyelem biztosítja a műszaki követelményeknek, és biztonsági okokból egységesen érvényesek felhőalapú környezetekben, mivel ezek a követelmények részletes.

> [!NOTE]
> Biztonsági alapkonfiguráció cégirányítási nem helyettesíti a meglévő informatikai csapatok, folyamatok és eljárások, amelyek a szervezete felhőben üzembe helyezett erőforrások védelme. Ezen a területen elsődleges célja, hogy üzleti biztonsági kockázatok azonosítása és kockázatcsökkentő útmutatást nyújt az informatikai munkatársak biztonsági infrastruktúra felelős. Ki, a cégirányítási szabályzatokat és folyamatok ügyeljen arra, hogy megfelelő IT-csoportoknak bevonja a tervezési és folyamatok ellenőrzése.

Ez a cikk ismerteti a módszer egy biztonsági alaptervet fegyelem tapasztalatlan a cégirányítási stratégia részeként. Az elsődleges Ez az útmutató célközönségét a szervezet felhőben dolgozó tervezők és más tagjai a Felhőbeli Cégirányítási csapat. Azonban a döntéseket hozhat, a házirendek és a folyamatokat, amelyek a fegyelem vet fel üzemeltetésicsoport és a hozzászólások megfelelő tagjaival, az informatikai RÉSZLEG és biztonsági csoportok, különösen azokat műszaki vezetői hálózatkezelés, végrehajtásáért felelős titkosítás és identitáskezelési szolgáltatásokat.

A megfelelő biztonsági döntések elősegítése érdekében fontos a sikeres a magánfelhők számára, és a szélesebb üzleti sikerek. Ha a szervezet nem rendelkezik a kiberbiztonsági szakértelem, fontolja meg a vonzó külső biztonsági tanácsadók a fegyelem összetevőjeként. Megfontolnia a vonzó [Microsoft Consulting Services programmal](https://www.microsoft.com/enterprise/services), a [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack/) bevezetési felhőszolgáltatásként vagy más külső felhő bevezetését szakértők vitatni a fegyelem kapcsolatos problémákat.

## <a name="policy-statements"></a>Házirend-utasítások

Gyakorlatban hasznosítható házirend-utasítások és az eredményül kapott architektúra követelmények alapját a biztonsági alapkonfiguráció fegyelem a szolgál. A házirend utasítás minták, olvassa el a cikk a [biztonsági alapterv házirend-utasítások](./policy-statements.md). Ezek a minták a szervezet cégirányítási szabályzatok kiindulási pontként szolgálhat.

> [!CAUTION]
> A mintául szolgáló házirendek közös felhasználói felületéről származnak. Jobban ezek a házirendek adott felhőbeli cégirányítási igényeihez igazítása, hajtsa végre az alábbi lépéseket, amelyek megfelelnek az egyedi üzleti házirend-utasítások létrehozásához szüksége van.

## <a name="developing-security-baseline-governance-policy-statements"></a>Fejlesztés a biztonsági alapkonfiguráció cégirányítási házirend-utasítások

Az alábbi hat lépést a példák és megfontolandó szempontokra biztonsági alapterv cégirányítási lehetséges módszert kínálnak. Kiindulási pontként minden lépés használata a Felhőbeli Cégirányítási csapaton belül, és az érintett üzleti hozzászólások informatikai és biztonsági csoportokat a szervezetben, a házirendek és a biztonsági kockázatok enyhítéséhez szükséges folyamatokat.

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
                        <h3>Biztonsági alapkonfiguráció sablon</h3>
                        <p class="x-hidden-focus">A biztonsági alapkonfiguráció fegyelem dokumentálja a sablon letöltése</p>
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
                        <p class="x-hidden-focus">Ismerje meg a okokra és a biztonsági alapkonfiguráció fegyelem informatikához kapcsolódó kockázatokat.</p>
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
                        <p class="x-hidden-focus">Megismerheti, hogy a megfelelő időben a biztonsági alapkonfiguráció fegyelem támogatásán mutatók.</p>
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
                        <p class="x-hidden-focus">A szabályzatoknak való megfelelés támogatása az alapvető biztonsági fegyelem javasolt folyamatokat.</p>
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
                        <p class="x-hidden-focus">Azure-szolgáltatások, amely támogatja a biztonsági alapkonfiguráció fegyelem implementálhatók.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>További lépések

Első lépésként egy adott környezetben üzleti kockázatot kiértékelésekor.

> [!div class="nextstepaction"]
> [Üzleti kockázatai ismertetése](./business-risks.md)
