---
title: 'CAF: Erőforrás konzisztencia motivációit és az üzleti kockázatok'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás konzisztencia motivációit és az üzleti kockázatok
author: alexbuckgit
ms.openlocfilehash: 19e0d761e4afa3473099bde2edc960c8b9eadb79
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298613"
---
# <a name="resource-consistency-motivations-and-business-risks"></a>Erőforrás konzisztencia motivációit és az üzleti kockázatok

Ez a cikk ismerteti az okokat, hogy ügyfeleink egy erőforrás konzisztencia területen belül a felhő adatirányítási stratégia általában fogad el. Néhány példa a lehetséges üzleti kockázatok, amelyek előmozdítják a házirend-utasítások is tartalmazza.

<!-- markdownlint-disable MD026 -->

## <a name="is-resource-consistency-relevant"></a>Az érintett erőforrás konzisztencia?

Esetén, erőforrások és számítási feladatok üzembe helyezéséhez, a felhő nagyobb rugalmasságot és kínál rugalmasságot leggyakrabban hagyományos helyszíni adatközpontokhoz. Azonban ezek lehetséges felhőalapú előnnyel is jár társalkalmazás is lehetséges kezelés hátrányai, amely is súlyosan veszélyeztetheti a felhőre való áttérés sikeres. Milyen eszközökre van telepített? Milyen csapatok saját milyen eszközöket? Kiegészítő számítási feladatok erőforrásokkal rendelkezik? Hogyan, hogy a megfelelő számítási feladatok esetén?

Győződjön meg arról, hogy erőforrásokat üzembe helyezett, frissítve, és következetesen és repeatably konfigurált, és, hogy a szolgáltatáskimaradásokat kis méretben fut és a lehető rövid idejű pótolja erőforrás konzisztencia elengedhetetlen.

Erőforrás konzisztencia tantárgy üzemidejére azonosítása, és a felhőbeli üzemelő példány részleteivel kapcsolatos üzleti kockázatot. Erőforrás konzisztencia tartalmazza az alkalmazások, számítási feladatok és az eszközintelligencia teljesítményének figyelését. A skálázási igények figyelembevételével készült, teljesítménytúllépéseknek szolgáltatói szerződés (SLA), és kerülje el proaktív módon SLA teljesítménytúllépéseknek automatikus szervizelés útján szükséges feladatok is tartalmaz.

Kezdeti tesztelési célú telepítéseket is szükség lehet lényegesen nagyobbra bevezetésével néhány szemléltetés elnevezési és szabványokat, így támogatja az erőforrás konzisztencia címkézés van szüksége. A felhőre való áttérés kiforrottabbá válik, és összetett, kritikus fontosságú eszközök telepítése, hogy az erőforrás konzisztencia tantárgy be kellene növekszik rövid idő alatt.

## <a name="business-risk"></a>Üzleti kockázat

Az erőforrás konzisztencia fegyelem próbál meg címet alapvető működési üzleti kockázatok. Az üzleti és informatikai csapata hozza létre az említett kockázatok azonosítása, és azok a relevancia alapján végzett figyelésére, tervezése és megvalósítása a felhőalapú környezetekben dolgozhat.

Kockázatok gyakori kockázatok használható kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, szervezet, de a következő szolgáltatása közötti eltérőek:

- **A szükségtelen üzemeltetési költségek**. Elavult vagy fel nem használt erőforrások és erőforrások vannak szükségesnél több erőforrással ellátott során igény alacsony, adja hozzá a szükségtelen működési költségeket.
- **Erőforrások underprovisioned**. Erőforrásokat, hogy a magasabb, mint az várható igény szerint a felhőbeli erőforrások vannak túlterhelte igény szerint üzletmenet megszakadása eredményezhet.
- **Felügyeleti hatékonysági**. Az egységes elnevezési és címkézési metaadatokat az erőforrásokhoz kapcsolódó hiánya problémákba források a felügyeleti feladatokat, vagy a tulajdonjoga és eszközökkel kapcsolatos számlázási információk azonosító informatikai munkatársak vezethet. Ez a felügyeleti hatékonysági, növelhető a költség- és a lassú informatikai válaszkészségének szolgáltatáskimaradás vagy egyéb működési problémákat eredményez.
- **Üzleti megszakítás**. Szolgáltatás megszakadását, hogy a szervezet megsértésének eredménye megállapított szolgáltatásiszint-szerződések (SLA) a vállalati üzleti vagy egyéb pénzügyi hatások elvesztését eredményezheti.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-üzleti kockázatok, amelyek valószínűleg vezeti be a jelenlegi cloud bevezetési tervet.

Ha valós üzleti kockázatai megismerése létrejött, a következő lépés az dokumentálni az üzleti szintű kockázatokat és a mutatók és a kulcs metrikákat kíván monitorozni a tolerancia.

> [!div class="nextstepaction"]
> [Megismerheti a mutatók, metrikákat és kockázattűrés](./metrics-tolerance.md)
