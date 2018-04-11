---
title: Hálózati DMZ
description: Az Azure-ban, hibrid rendszer részeként futó alkalmazásoknak és összetevőknek a jogosulatlan behatolásokkal szembeni védelmére szolgáló rendelkezésre álló különböző metódusokat ismerteti és hasonlítja össze.
layout: LandingPage
ms.openlocfilehash: 98df0a25767c7a7282e67381c6465fe3263ce1fa
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="network-dmz"></a>Hálózati DMZ

Ezek a referenciaarchitektúrák a hálózati DMZ-knek az Azure virtuális hálózat és a helyszíni hálózat vagy az internet közötti határ védelmére történő létrehozásával kapcsolatos ajánlott eljárásokat mutatják be.

<section class="series">
    <ul class="panelContent">
    <!-- DMZ between Azure and on-premises -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-hybrid.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Szegélyhálózat (DMZ) az Azure és a helyszíni hely között</h3>
                        <p>Egy biztonságos hibrid hálózatot valósít meg, amely kiterjeszti a helyszíni hálózatot az Azure-ba.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- DMZ between Azure and the Internet -->
<li style="display: flex; flex-direction: column;">
    <a href="./secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Szegélyhálózat (DMZ) az Azure és az internet között</h3>
                        <p>Egy biztonságos hálózatot valósít meg, amely engedélyezi az internetes forgalmat az Azure-ba.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>