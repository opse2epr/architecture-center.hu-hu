---
title: Mikroszolgáltatások létrehozása az Azure-ban
description: Mikroszolgáltatási architektúrák tervezése, létrehozása és működtetése az Azure-ban
ms.date: 03/07/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: e74d6f6098eb68c8bbd737cf3a047ce5de04327d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346144"
---
# <a name="building-microservices-on-azure"></a>Mikroszolgáltatások létrehozása az Azure-ban

<!-- markdownlint-disable MD033 -->

<img src="../_images/microservices.svg" style="float:left; margin-top:8px; margin-right:8px; max-width: 80px; max-height: 80px;"/>

A mikroszolgáltatás egy népszerű architekturális stílus a rugalmas, hatékonyan méretezhető, függetlenül üzembe helyezhető és gyorsan fejleszthető alkalmazások létrehozásához. De a sikeres mikroszolgáltatási architektúrához új megközelítés szükséges az alkalmazások tervezésekor és létrehozásakor.

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./introduction.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Mik azok a mikroszolgáltatások?</h3>
                        <p>Miben különböznek a mikroszolgáltatások a többi architektúrától, illetve mikor érdemes használni őket?</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../guide/architecture-styles/microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Mikroszolgáltatási architektúrastílus</h3>
                        <p>A mikroszolgáltatási architektúrastílus magas szintű áttekintése</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="examples-of-microservices-architectures"></a>Példák mikroszolgáltatási architektúrákra

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/infrastructure/service-fabric-microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>A Service Fabric használata monolitikus alkalmazások felbontásához</h3>
                        <p>Az ASP.NET-webhelyek mikroszolgáltatásokra való felbontásának iteratív megközelítése.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/data/ecommerce-order-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Skálázható rendelésfeldolgozás az Azure-ban</h3>
                        <p>Rendelésfeldolgozás mikroszolgáltatásokkal implementált funkcionális programozási modellel.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="build-a-microservices-application"></a>Mikroszolgáltatás-alkalmazás készítése

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./model/domain-analysis.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Mikroszolgáltatások modellezése tartományelemzéssel</h3>
                        <p>A mikroszolgáltatások tervezésekor a gyakori buktatók elkerülése érdekében határozza meg a mikroszolgáltatás határait tartományelemzés segítségével.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Az Azure Kubernetes Services (AKS) referenciaarchitektúrája</h3>
                        <p>Ez a referenciaarchitektúra egy olyan alapvető AKS-konfigurációt mutat be, amely a legtöbb üzemelő példány kiindulópontjaként szolgálhat.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Az Azure Service Fabric referenciaarchitektúrája</h3>
                        <p>Ez a referenciaarchitektúra olyan javasolt konfigurációt mutat be, amely a legtöbb üzemelő példány kiindulópontjaként szolgálhat.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Mikroszolgáltatási architektúra tervezése</h3>
                        <p>Ezek a cikkek részletesen bemutatják, hogyan készíthet mikroszolgáltatás-alkalmazást az Azure Kubernetes Servicest (AKS) használó referenciaimplementáció alapján.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/patterns.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Tervezési minták</h3>
                        <p>Hasznos tervezési minták mikroszolgáltatásokhoz.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="operate-microservices-in-production"></a>A mikroszolgáltatások működtetése éles környezetben

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./logging-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Naplózás és figyelés</h3>
                        <p>A mikroszolgáltatási architektúrák elosztott jellege miatt különösen fontos a naplózás és a monitorozás.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./ci-cd.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Folyamatos integráció és üzembe helyezés</h3>
                        <p>A folyamatos integráció és teljesítés (CI/CD) kulcsfontosságú a mikroszolgáltatások sikeréhez.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>
