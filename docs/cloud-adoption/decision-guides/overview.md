---
title: 'CAF: Architektúrával kapcsolatos döntés útmutatók'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a felhő bevezetésének keretrendszer architektúrával kapcsolatos döntés útmutatók.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899362"
---
# <a name="architectural-decision-guides"></a>Architektúrával kapcsolatos döntés útmutatók

Az architektúrával kapcsolatos döntés útmutatók a felhő bevezetésének keretrendszer írja le, minták és a modellek, amelyek segítenek a felhőalapú cégirányítási tervezési útmutatást létrehozásakor. Minden döntés útmutató egy alapvető infrastruktúra összetevője magánfelhők koncentrál, és megjeleníti a lehetséges minták vagy modellekkel támogatják az adott felhő központi telepítési forgatókönyv.

Amikor hozzákezd a szervezete felhőben cégirányítási létesíteni, gyakorlatban hasznosítható cégirányítási Journey egy alapkonfiguráció ütemterv adja meg. Ezek az utak legyen azonban feltételezéseket követelményeknek és a prioritások, amelyek nem tükrözik azokat a szervezet.
Döntés az útmutatók a minta cégirányítási Journey kiegészítik azáltal, hogy helyettesítő mintákat, és modellek, amelyek segítenek a példa-tervezési segédlet a saját igényeinek megfelelően az architekturális tervezés kiválasztott lehetőségeket igazítása.

## <a name="design-guidance-categories"></a>Tervezési útmutatást kategóriák

A következő kategóriák jelöli, hogy a felhő központi telepítések egy alapvető technológia. Az egyes választott a példa tervezési Journey, amely nem felel meg a követelményeknek vizsgálja meg a beállításokat alább vonatkozó részében egy minta vagy a modell jobban az igényeinek megfelelő kiválasztása.

[Előfizetések](./subscriptions/overview.md): Tervezze meg a felhőbeli üzemelő példány előfizetés tervezési és a fiók tulajdonjogát, a számlázásra és a felügyeleti funkciókat a szervezet megfelelő struktúra.

[identitás](./identity/overview.md): Felhőalapú identitás-szolgáltatások integrálható a meglévő identitás erőforrások hozzáférés-vezérlés az informatikai környezetben támogatásához.

[A házirend betartatása](./policy-enforcement/overview.md): Határozzon meg, és az erőforrások és a felhőben üzembe helyezett számítási feladatok a szervezeti házirend szabályok érvényesítése.

[Erőforrás konzisztencia](./resource-consistency/overview.md): Győződjön meg arról, hogy üzembe helyezés és a szervezet a felhőalapú erőforrások igazítása kényszerítése erőforrás felügyeleti és a házirend követelményeinek.

[Erőforrás-címkézési](./resource-tagging/overview.md): A számlázási modellekkel, felhő számlázási megközelítések, felügyeleti, hozzáférés-vezérlés és működési hatékonyságát a felhőalapú erőforrások rendezéséhez. Erőforrás-címkézési szükséges egy egységes és jól szervezett elnevezési és metaadatok sémát.

[Szoftveralapú hálózatok](./software-defined-network/overview.md): Gyors üzembe helyezési és módosítása a virtualizált hálózati funkciói segítségével a felhőbe biztonságos számítási feladatok üzembe helyezéséhez. Szoftveralapú hálózat (SDNs) támogatja a rugalmas munkafolyamatok, erőforrások elkülönítése, és felhőalapú rendszerek integrálását az meglévő IT-infrastruktúrája.

[Titkosítási](./encryption/overview.md): A titkosítással, fontos szempont a biztonsági belül üzembe helyezése a felhőben tárolt bizalmas adatok védelme.

[Naplók és a jelentéskészítés](./log-and-report/overview.md): Felhőalapú erőforrások által létrehozott figyelő naplóadatokat. Adatok elemzése betekintést állapotát az operatív, a karbantartással és a számítási feladatok házirend kényszerítési állapotát.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan előfizetések és fiókok erre a célra a felhőbeli üzembe helyezés alap.

> [!div class="nextstepaction"]
> [Előfizetések kialakítása](subscriptions/overview.md)
