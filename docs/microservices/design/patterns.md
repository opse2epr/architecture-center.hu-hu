---
title: Tervezési minták a mikroszolgáltatások
description: Tervezési minták, amely hatékony mikroszolgáltatási architektúra megvalósítása.
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 8cdbf95c753770910fa4a94384c9809db9fda746
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298834"
---
# <a name="design-patterns-for-microservices"></a>Tervezési minták a mikroszolgáltatások

A mikroszolgáltatások célja decomposing kis autonóm servicesbe, egymástól függetlenül telepíthető az alkalmazás által a alkalmazás kiadások, a sebesség növelése érdekében. A mikroszolgáltatási architektúra is elérhetővé teszi a áttekinthet néhány problémát. Az itt látható tervezési minták segítségével, ezek a kihívások megoldásához.

![A Mikroszolgáltatások tervezési minták](../images/microservices-patterns.png)

[**Nagykövet** ](../../patterns/ambassador.md) kiszervezheti a gyakori ügyfélkapcsolati feladatok, például a figyelés, naplózás, Útválasztás és biztonság (pl. TLS) nyelven használható független módon. Nagykövet-szolgáltatások gyakran települnek oldalkocsiként (lásd alább).

[**Sérülésgátló réteg** ](../../patterns/anti-corruption-layer.md) valósít meg egy a adapterréteget új és örökölt alkalmazások között, annak érdekében, hogy egy új alkalmazás kialakítása nem korlátozódik a régi rendszerektől való függőségek.

[**Háttérrendszerek és Előtérrendszerek** ](../../patterns/backends-for-frontends.md) hoz létre a különböző típusú ügyfelek, például az asztali és mobil külön háttérrendszerekhez. Ezzel a módszerrel az egyetlen háttérszolgáltatás ütköző igényeit kielégítő különböző ügyfél kezeléséhez nem szükséges. Ez a minta segíthet használni, hogy mindegyik mikroszolgáltatás egyszerű, elválasztó ügyfelekre vonatkozik.

[**Válaszfal** ](../../patterns/bulkhead.md) különíti el a kritikus fontosságú erőforrások, például kapcsolatkészletben, a memória és a Processzor, a számítási feladat vagy szolgáltatás. Válaszfalak használatával egy egyetlen számítási feladat (vagy szolgáltatás) nem lehet felhasználni mások starving erőforrásokat. Ezt a mintát növeli a rugalmasságot, a rendszer megakadályozza, hogy egy szolgáltatás által okozott hiba egész hibasorozatot indíthat.

[**Átjáró-összesítési** ](../../patterns/gateway-aggregation.md) összesíti az egyes mikroszolgáltatásokat több kérelmeket egyetlen kérelemmé, csökkentve a fogyasztók és a szolgáltatások közötti forgalmat.

[**Átjáró Átjárókiürítés** ](../../patterns/gateway-offloading.md) lehetővé teszi, hogy mindegyik mikroszolgáltatás kiszervezheti a megosztott szolgáltatás funkciókat, például SSL-tanúsítványokat, API-átjáró használatát.

[**Átjáró útválasztási** ](../../patterns/gateway-routing.md) irányítja a kérelmeket, egyetlen végpont használatával több mikroszolgáltatásból, hogy a felhasználók számos különböző végpontok kezelése nem szükséges.

[**Az oldalkocsi** ](../../patterns/sidecar.md) helyez üzembe egy alkalmazás egy külön tároló vagy folyamat elkülönítést és beágyazást biztosíthat biztosít segítő összetevőit.

[**Leépítő** ](../../patterns/strangler.md) támogatja a növekményes újrabontás egy alkalmazás oly módon, hogy egyes funkciódarabokat új szolgáltatásokkal.

A felhőkialakítási minták az Azure Architecture Centert a kész katalógust, lásd: [tervezési minták Felhőkhöz](../../patterns/index.md).
