---
title: "Helyszíni hálózat csatlakoztatása az Azure-hoz"
description: "Ajánlott architektúrák a helyszíni hálózatok és az Azure közötti biztonságos, robusztus hálózati kapcsolatokhoz."
layout: LandingPage
ms.openlocfilehash: 0707d17295e338af0176bd0cea806615ef05f9ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure"></a>Helyszíni hálózat csatlakoztatása az Azure-hoz

Ezek a referenciaarchitektúrák a helyszíni hálózat és az Azure közötti robusztus hálózati kapcsolat létrehozásához mutatnak bevált gyakorlatokat. [Melyiket válasszam?](./considerations.md)

<ul class="panelContent">
    <li>
        <a href="./vpn.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/vpn.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN</h3>
                            <p>A helyszíni hálózatot kiterjesztheti az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute</h3>
                            <p>Helyszíni hálózat kiterjesztése az Azure-ra az Azure ExpressRoute segítségével</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute-vpn-failover.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute VPN-alapú feladatátvétellel</h3>
                            <p>Helyszíni hálózat kiterjesztése az Azure-ra az Azure ExpressRoute segítségével, VPN-nel feladatátvételi kapcsolatként.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./hub-spoke.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/hub-spoke.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Küllős topológia</h3>
                            <p>Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye. A küllők az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók. </p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
</ul>

