---
title: 'CAF: Biztonsági alapkonfiguráció motivációit és az üzleti kockázatok'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapkonfiguráció motivációit és az üzleti kockázatok
author: BrianBlanchard
ms.openlocfilehash: 8407ed358e5862e466176096ee6a82ad792027cb
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299363"
---
# <a name="security-baseline-motivations-and-business-risks"></a>Biztonsági alapkonfiguráció motivációit és az üzleti kockázatok

Ez a cikk ismerteti az okokat, hogy ügyfeleink a biztonsági alapkonfiguráció fegyelem belül a felhő adatirányítási stratégia általában fogad el. Néhány példa a lehetséges üzleti kockázatok, amelyek előmozdítják a házirend-utasítások is tartalmazza.

<!-- markdownlint-disable MD026 -->

## <a name="is-a-security-baseline-relevant"></a>Fontos biztonsági alapterv?

Biztonsági legfontosabb szempont minden informatikai szervezet. Magánfelhők között számos, mint a hagyományos helyszíni adatközpontokban üzemeltetett számítási feladatokat ugyanazon biztonsági kockázatokat. Azonban hiányoznak a tárolására, és a számítási feladatokat futtató fizikai hardvereket közvetlen tulajdonjogát a nyilvános felhőalapú platformon, jellege azt jelenti, a cloud security igényel saját szabályzat és a folyamatok.

Egyik elsődleges dolog, hogy set felhőbeli biztonsági cégirányítási szereplőkkel a hagyományos biztonsági házirend-e az egyszerű, amelyekkel erőforrásokat lehet létrehozni, potenciálisan biztonsági rések hozzáadása, ha biztonsági nincs üzembe helyezés előtt. A rugalmasságot, amelyet a technológiák, például [szoftveralapú hálózatkezelés (SDN)](../../decision-guides/software-defined-network/overview.md) jelentenek a gyorsan változó a felhőalapú hálózati topológiáról is egyszerűen módosíthatja a teljes hálózat támadási felületét előre nem látható módon. Felhőalapú platformon is biztosítanak, eszközökről és szolgáltatásokról, a biztonsági lehetőségei a módszerekkel nem mindig lehetséges, a helyszíni környezetben.

Az összeg fektet be biztonsági házirend, és a folyamatok jelentős függ a felhőbeli üzemelő példány jellegét. Kezdeti tesztkörnyezetek csak szükség lehet a lehető legegyszerűbb a biztonsági házirendeket, amíg egy alapvető fontosságú számítási feladat magában foglalja a címzési összetett és széles körű biztonsági igényeinek. Központi telepítések kell bizonyos szintű tantárgy érhet el.

A biztonsági alapkonfiguráció fegyelem tartalmazza a vállalati szabályzatoknak, és manuális dolgozza fel, hogy is nyilvántartani védik a felhőbeli üzemelő példány biztonsági kockázatokkal szemben.

> [!NOTE]
>Fontos tudni, bár [identitásra építve](../identity-baseline/overview.md) biztonsági alapterv, illetve, hogy hozzáférés-vezérlés, hogyan viszonyul kontextusában a [öt oktatnak, felhőalapú Cégirányítási](../overview.md) ki [ Identitásra építve](../identity-baseline/overview.md) saját szabályok betartását, mint önálló biztonsági alaptervből.

## <a name="business-risk"></a>Üzleti kockázat

A biztonsági alapkonfiguráció fegyelem próbál meg címet alapvető üzleti biztonsági kockázatokat. Az üzleti kockázatok azonosítása, és azok a relevancia alapján végzett figyelésére, tervezése és megvalósítása a felhőalapú környezetekben dolgozhat.

Kockázatok közös biztonsági kockázatok használható kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, szervezet, de a következő szolgáltatása közötti eltérőek:

- **Adatok illetéktelen behatolás**. Ügyfelek, szerződéses problémák vagy jogi következményekkel elvesztése nem szándékos kitettség vagy a felhőben üzemeltetett bizalmas adatok elvesztése vezethet.
- **A szolgáltatás megszűnésének**. Kimaradások és más teljesítménnyel kapcsolatos problémák miatt nem biztonságos infrastruktúra megzavarja a normál működés, és a hatékonyság vagy elveszett üzleti.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-üzleti kockázatok, amelyek valószínűleg vezeti be a jelenlegi cloud bevezetési tervet.

Ha valós üzleti kockázatai megismerése létrejött, a következő lépés az dokumentálni az üzleti szintű kockázatokat és a mutatók és a kulcs metrikákat kíván monitorozni a tolerancia.

> [!div class="nextstepaction"]
> [Megismerheti a mutatók, metrikákat és kockázattűrés](./metrics-tolerance.md)
