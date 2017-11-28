---
title: "Azure-referenciaarchitektúrák"
description: "Referenciaarchitektúrák, tervek és részletes megvalósítási útmutatók az Azure-ban végzett gyakori számítási feladatokhoz."
layout: LandingPage
ms.openlocfilehash: eaff531c28faeb0aac774acf70d4334fb4e1f319
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="azure-reference-architectures"></a>Azure-referenciaarchitektúrák

A referenciaarchitektúrák forgatókönyv szerint vannak elrendezve, az egymáshoz kapcsolódó architektúrák pedig csoportosítva vannak. Minden architektúra tartalmaz ajánlott eljárásokat, valamint méretezhetőségre, rendelkezésre állásra, kezelhetőségre és biztonságra vonatkozó megfontolandó szempontokat. A legtöbb architektúra emellett üzembe helyezhető megoldást is tartalmaz.

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
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
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
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

<ul class="panelContent cardsI">
<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
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

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
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


</section>

