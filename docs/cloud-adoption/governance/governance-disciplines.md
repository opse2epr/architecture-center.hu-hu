---
title: 'CAF: A felhőszabályozás öt szemlélete'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Cégirányítási tartalmakra CAF áttekintése
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ddd7f5e4b7e79ebe2ddf87be7421b6c4691aced1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639869"
---
# <a name="the-five-disciplines-of-cloud-governance"></a>A felhőszabályozás öt szemlélete

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Üzleti folyamatok és a technológiai platform bármilyen változás kockázati mutatja be. Felhőalapú Cégirányítási csapatok, amelynek tagjai is ismertek felhőalapú jegyektől, minimális megszakítással használhatja tovább elfogadását, vagy az innováció eszköztárát kockázatok csökkentése is biztosítja.<br/><br/>A CAF cégirányítási modell ezek a döntések (függetlenül a választott felhőalapú platform) végigvezeti a fejlesztését a vállalati házirenddel való összpontosítással és <a href="#disciplines-of-cloud-governance">felhőalapú Cégirányítási oktatnak</a>. <a href="./journeys/overview.md">Gyakorlatban hasznosítható Journey</a> bemutatása az Azure-szolgáltatások használata ebben a modellben. Ez a cikk egy kezdőlapja a CAF cégirányítási modell öt állapítják funkcionál.
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
<i>1. ábra Vállalati szabályzatok és a felhő Cégirányítási öt oktatnak ábrája</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="disciplines-of-cloud-governance"></a>Felhőalapú irányítási oktatnak

Minden felhőszolgáltató között vannak közös cégirányítási oktatnak, amely tájékoztatja a házirendek és eszközlánccal való igazítása útmutató alapul szolgálhat. Ezek a szabályok megfelelő szintű automation és a vállalati szabályzat kényszerítésének kapcsolatos döntések útmutató felhőszolgáltatók között.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsA">
<li style="display: flex; flex-direction: column;">
    <a href="./cost-management/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/cost-management.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cost Management</h3>
                        <p>Egy elsődleges szempont felhőfelhasználók számára. Fejlesztésére költségvezérlés az összes felhőalapú platformra vonatkozó házirendek.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./security-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/security-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Biztonsági alapkonfiguráció</h3>
                        <p>Egy összetett és személyes témakör egyedi minden cégnek. Biztonsági követelmények létrejöttük felhőalapú cégirányítási szabályzatokat és kényszerítési vonatkozik, ezek a követelmények több hálózati, adatok és az eszközintelligencia konfigurációkban.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./identity-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/identity-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Identitáskezelési alapkonfiguráció</h3>
                        <p>Identitással kapcsolatos követelmények alkalmazását lévő inkonzisztenciák növelheti az illetéktelen behatolás kockázatát. Az identitásra építve fegyelem módja annak, identitás következetesen alkalmazzák között a felhő bevezetésének erőfeszítések összpontosít.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./resource-consistency/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/resource-consistency.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Erőforrás-konzisztencia</h3>
                        <p>Felhőbeli műveletek attól függ, hogy az erőforrás-konfigurációban konzisztencia. Keresztül cégirányítási eszközök, erőforrások konzisztens módon konfigurálhatja az előkészítési, eltéréseket, felfedezhetősége és helyreállítási kockázatok csökkentése érdekében.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./deployment-acceleration/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/deployment-acceleration.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Üzembe helyezés gyorsítása</h3>
                        <p>Forrásadattárakból szabványügyi szervezet és a telepítési és konfigurációs módszer konzisztencia javítása eljárásaink. Eszközök felhőalapú cégirányítási érhetők el, ha azok hozzon létre egy felhőalapú tényező, amely meggyorsíthatja a telepítési műveletek.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->
