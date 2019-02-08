---
title: 'CAF: Motivációit és az üzleti kockázatok, a meghajtó telepítési gyorsítás'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a felhő adatirányítási stratégia részeként üzembe helyezési gyorsulás tantárgy.
author: alexbuckgit
ms.openlocfilehash: 854b1fd420de605a665922e9b207e6aecbfab2f0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898754"
---
# <a name="deployment-acceleration-motivations-and-business-risks"></a>Üzembe helyezési gyorsítás motivációit és üzleti kockázatai

Ez a cikk ismerteti az okokat, hogy ügyfeleink egy központi telepítési gyorsítás fegyelem belül a felhő adatirányítási stratégia általában fogad el. Emellett biztosítja az üzleti kockázatok néhány példa adott meghajtó házirend-utasítások.

<!-- markdownlint-disable MD026 -->

## <a name="is-deployment-acceleration-relevant"></a>Fontos gyorsítás üzembe helyezési?

A helyszíni rendszerek gyakran eredeti képek vagy telepítési szkriptek használatával helyezi üzembe. További konfigurációs általában szükség, amelyek több lépést vagy emberi beavatkozás járhat. Ezeket a manuális folyamatokat sok hibalehetőséget rejtő és gyakran "konfigurációs csúszásokat", időigényes hibaelhárítás és javítás feladatokat igénylő eredményez.

A legtöbb Azure-erőforrások telepíthető, és manuálisan az Azure Portalon konfigurálni. Ez a megközelítés elegendő lehet az igényeinek, ha csak néhány erőforrások kezeléséhez rendelkeznie. A felhő hagyatéki növekedésével azonban a szervezet automatikus kell kihasználásához a skálázás, a feladatátvétel és a vészhelyreállítás Azure által biztosított lehetőségek a felhőbeli erőforrások központi telepítése. A fejlesztési és üzemeltetési vagy DevSecOps megközelítés bevezetése legtöbbször a legjobb módszer a központi telepítések felügyeletéhez szükséges.

Nagy teljesítményű üzembe helyezési gyorsítás csomag biztosítja, hogy a felhőbeli erőforrások vannak telepítve, frissül, és helyesen és következetesen legyenek konfigurálva és így marad. Az adatbázisplatform működését is az üzembe helyezés gyorsítás stratégia akkor is lényeges a [Cost Management stratégia](../cost-management/overview.md). Automatikus üzembe helyezést és a felhőbeli erőforrások konfigurációját lehetővé teszi a különböző méretezési műveletek, vagy szabadítsa fel erőforrásokat, ha igény alacsony vagy időhöz kötött, így a számlázás a feladatonként az erőforrások igény szerint.

## <a name="business-risk"></a>Üzleti kockázat

A központi telepítési gyorsítás fegyelem próbál oldja meg a következő üzleti kockázatok. Során a felhőre való áttérés figyelheti a relevancia alapján végzett a következő:

- **A szolgáltatás megszűnésének**. Kiszámítható megismételhető üzembe helyezési hiánya dolgozza fel, vagy nem felügyelt módosításait a rendszer konfigurációját megzavarhatja a normál működés, és hatékonyság vagy elveszett üzleti.
- **Költség meghaladása**. A konfigurációban az erőforrásokat a váratlan módosítások győződjön meg arról is, problémák eredendő okát bonyolultabb, a fejlesztési, műveleti és karbantartási költségek előléptetése azonosítása.
- **Szervezeti hatékonysági**. Fejlesztési, műveleteket és biztonsági csapatok közötti akadályok, hatékony bevezetési a felhőalapú technológiák számos kihívást és a egy egységes felhőalapú cégirányítási modell fejlesztésének okozhat.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-üzleti kockázatok, amelyek valószínűleg vezeti be a jelenlegi cloud bevezetési tervet.

Ha valós üzleti kockázatai megismerése létrejött, a következő lépés az dokumentálni az üzleti szintű kockázatokat és a mutatók és a kulcs metrikákat kíván monitorozni a tolerancia.

> [!div class="nextstepaction"]
> [Metrikák, a mutatók és a szervezet kockázattűrési határát](./metrics-tolerance.md)
