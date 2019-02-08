---
title: 'CAF: Nagyvállalati - kezdeti vállalati szabályzat a adatirányítási stratégia mögött'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: Nagyvállalati - kezdeti vállalati szabályzat a adatirányítási stratégia mögött.
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899387"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a>Nagyvállalati: A cégirányítási stratégia mögött kezdeti vállalati szabályzat

A következő vállalati szabályzat határozza meg a kezdeti cégirányítási pozíció, amely a kiindulási pontként tartson a folyamatban. Ez a cikk a kockázatok korai szakaszában, a kezdeti házirend-utasítások és a házirend-utasítások végrehajtásához korai folyamatok határozza meg.

> [!NOTE]
>A vállalati szabályzat nem technikai dokumentum, de azt meghajtók számos technikai döntéseket hozhat. A cégirányítási MVP ismertetett a [áttekintése](./overview.md) végső soron Ez a szabályzat származik. A cégirányítási MVP-k megvalósítása előtt a szervezet kell kidolgozni a vállalati házirend alapján saját célkitűzések és az üzleti kockázatok.

## <a name="cloud-governance-team"></a>A felhő Cégirányítási csapata

A CIO nemrég tartott egy értekezlet megértéséhez a személyazonosításra alkalmas adatok és a kritikus fontosságú házirendek előzményeit, és tekintse át ezeket a szabályzatokat módosításának hatása az informatikai szabályozással csapatával. Ő is olvashat a teljes lehetséges a felhőben az informatikai és a vállalat.

A konferencia után az informatikai szabályozással csoport két tagjára kutatási és támogatja a felhő tervezése erőfeszítések engedélyt kért. FELISMERVE kell irányítási és informatikai árnyék-infrastruktúra korlátozása lehetőséghez, Informatikai igazgató Cégirányítási támogatja ezt az elképzelést. Az adott a felhő Cégirányítási csapat született. A következő néhány hónapban, a karbantartás során feltárása a felhőben a cégirányítási szempontjából sok a hiba, öröklik. Ez kap őket a Felhőbeli jegyektől a kézjegy. A későbbi fejlesztések folyamaton jelennek meg hogyan azok a szerepkörök az idő előrehaladtával változik.

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a>Tolerancia mutatók

Az aktuális kockázattűrése magas és a cégirányítási felhőalapú infrastruktúrán végezhetjük a képességéhez értéke alacsony. Ennek megfelelően a tolerancia mutatók járhat el egy korai figyelmeztető rendszert időt és energiát, befektetés aktiválásához. Ha a következő mutatók jelennek meg, célszerű előre megtervezni a adatirányítási stratégia fejlődnek lenne.

- A Cost Management: Méretezés a központi telepítés meghaladja a felhőbe 1000 eszközök, vagy havi kiadások meghaladja a 10 000 USD havonta.
- Identitásra építve: Belefoglalási örökölt vagy harmadik felek többtényezős hitelesítés (MFA) követelményekkel rendelkező alkalmazások.
- Biztonsági alapkonfiguráció: Védett adatok meghatározott cloud bevezetési terveket felvételét.
- Erőforrás konzisztencia: Meghatározott cloud bevezetési terveket alapvető fontosságú alkalmazásokat felvételét.

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a>További lépések

A vállalati házirend előkészíti a felhő Cégirányítási csapat megvalósításához a cégirányítási MVP, amely bevezetésére alapját fogják adni. A következő lépés, hogy az MVP megvalósításához.

> [!div class="nextstepaction"]
> [Bevált gyakorlat ismertetése](./best-practice-explained.md)