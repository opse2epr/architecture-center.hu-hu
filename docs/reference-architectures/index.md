---
title: Azure-referenciaarchitektúrák
description: Referenciaarchitektúrák, tervek és részletes megvalósítási útmutatók az Azure-ban végzett gyakori számítási feladatokhoz.
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 9708c975e543a04cafe22f38194e833be44d7b81
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Azure-referenciaarchitektúrák

A referenciaarchitektúrák forgatókönyv szerint vannak elrendezve, az egymáshoz kapcsolódó architektúrák pedig csoportosítva vannak. Minden architektúra tartalmaz ajánlott eljárásokat, valamint méretezhetőségre, rendelkezésre állásra, kezelhetőségre és biztonságra vonatkozó megfontolandó szempontokat. A legtöbb architektúra emellett üzembe helyezhető megoldást is tartalmaz.

<section class="series">
    <ul class="panelContent">
    <!-- Windows VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-windows/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Windows rendszerű virtuális gépek számítási feladatai</h3>
                        <p>Ez a sorozat először egyetlen Windows rendszerű, majd több, elosztott terhelésű virtuális gép, és végül egy többrégiós N szintű alkalmazás futtatására vonatkozó ajánlott eljárásokat ismerteti.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Linux VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-linux/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Számítási feladatok Linux rendszerű virtuális gépeken</h3>
                        <p>Ez a sorozat először egyetlen Linux rendszerű, majd több, elosztott terhelésű virtuális gép, és végül egy többrégiós N szintű alkalmazás futtatására vonatkozó ajánlott eljárásokat ismerteti.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hybrid network -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Hibrid hálózat</h3>
                        <p>Ez a sorozat a helyszíni hálózat és az Azure közötti hálózati kapcsolat létrehozására vonatkozó beállításokat ismerteti.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Network DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Hálózati DMZ</h3>
                        <p>Ez a sorozat azt ismerteti, hogyan hozhat létre hálózati DMZ-t az Azure virtuális hálózat és a helyszíni hálózat vagy az internet közötti határ védelmére.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Identity management -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./identity/images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Identitáskezelés</h3>
                        <p>Ez a sorozat a helyszíni Active Directory (AD) környezet Azure-hálózattal való integrálásának lehetőségeit ismerteti.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- App Service web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>App Service webalkalmazás</h3>
                        <p>Ez a sorozat az Azure App Service szolgáltatást használó webalkalmazások ajánlott eljárásait ismerteti.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
    <!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./jenkins/images/logo.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Jenkins-buildkiszolgáló</h3>
                        <p>Skálázható, nagyvállalati szintű Jenkins-kiszolgáló helyezhet üzembe és üzemeltethet az Azure-ban.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sharepoint/images/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SharePoint Server 2016-farm</h3>
                        <p>Magas rendelkezésre állású SharePoint Server 2016-farm üzembe helyezése és futtatása az Azure-ban SQL Server Always On rendelkezésre állási csoportokkal.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SAP NetWeaver and SAP HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver és SAP HANA</h3>
                        <p>A SAP NetWeaver és SAP HANA üzembe helyezése és futtatás magas rendelkezésre állású környezetben az Azure-ban.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>