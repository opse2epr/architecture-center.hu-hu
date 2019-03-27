---
title: 'CAF: Identitás alapkonfiguráció motivációit és üzleti kockázatai'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás alapkonfiguráció motivációit és üzleti kockázatai
author: BrianBlanchard
ms.openlocfilehash: ef831d1b3b01251b27f1f7b18e551c49996a2468
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299284"
---
# <a name="identity-baseline-motivations-and-business-risks"></a>Identitás alapkonfiguráció motivációit és üzleti kockázatai

Ez a cikk ismerteti az okokat, hogy ügyfeleink az identitásra építve területen belül a felhő adatirányítási stratégia általában fogad el. Emellett biztosítja az üzleti kockázatok néhány példa adott meghajtó házirend-utasítások.

<!-- markdownlint-disable MD026 -->

## <a name="is-identity-baseline-relevant"></a>Az identitásra építve megfelelő?

A hagyományos helyszíni címtárak lehetővé teszi a cégek számára az szigorúan szabályozhatja az engedélyeket és a házirendek felhasználók, csoportok és szerepkörök belül adatközpontok, valamint a belső hálózatok számára tervezték. Általában ez célja, hogy egyetlen bérlő megvalósításokhoz, csak a helyszíni környezetben alkalmazható szolgáltatásokkal támogatják.

Felhőbeli identitás-szolgáltatások célja, hogy bontsa ki a szervezet hitelesítési és hozzáférés-vezérlő képességeit az internetre. Akkor támogatja a több-bérlős, és felhasználók kezelése és hozzáférés-házirend a felhőalapú alkalmazások és központi telepítések segítségével. Nyilvános felhő platformok natív felhőalapú identitás-szolgáltatások kiegészítő felügyeleti és üzembe helyezési feladatokat valamilyen formában, és képesek [különböző az integráció szintje különböző](../../decision-guides/identity/overview.md) együtt a meglévő helyszíni identitáskezelési megoldások. Ezek a Funkciók, mint a hagyományos helyszíni megoldások megkövetelése az több összetett identitás felhőhasználatiszabályzat eredményezhet.

Az identitásra építve fegyelem, a felhőbeli üzemelő példány lesz a csoport méretétől függ, és a egy meglévő felhőalapú identitás-megoldását integrálnia kell fontossága a helyszíni identitáskezelési szolgáltatása. Kezdeti tesztelési célú telepítéseket is szükség lehet nagy átszállítást megakadályozzák a felhasználó munkahelye vagy felügyeleti, de, a felhő hagyatéki kiforrottá, valószínűleg szüksége lesz egyre összetettebb szervezeti integráció és a központi felügyelet támogatásához.

## <a name="business-risk"></a>Üzleti kockázat

Az identitásra építve fegyelem próbál meg címet identity services és a hozzáférés-vezérlés kapcsolatos alapvető üzleti kockázatok. Az üzleti kockázatok azonosítása, és azok a relevancia alapján végzett figyelésére, tervezése és megvalósítása a felhőalapú környezetekben dolgozhat.

Kockázatok közös identitással kapcsolatos kockázatokat használható kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, szervezet, de a következő szolgáltatása közötti eltérőek:

- **Jogosulatlan hozzáféréstől**. Bizalmas adatok és erőforrások a jogosulatlan felhasználók elérhető adatszivárgás vagy szolgáltatáskimaradás, a szervezet biztonsági határt megsértése, és megelőzheti a cég vagy jogi kötelezettség és vezethet.
- **Több identitáskezelési megoldások miatt elégtelenségekkel**. Több identitás szolgáltatások bérlővel rendelkező szervezeteknek több fiókot lehet szükség, a felhasználók számára. Ezzel a módszerrel a felhasználók számára, akik több készletek és a hitelesítő adatok megjegyzése kell elégtelenségekkel informatikai a fiókok kezelése több rendszer között. Hozzáférési hozzárendelések felhasználó identitáskezelési megoldások között nem frissíti a személyzet, a teams és üzleti célok módosítása is, ha a felhőbeli erőforrások sérülékennyé válhat a jogosulatlan hozzáférést vagy a felhasználók nem fér hozzá a szükséges erőforrásokat.
- **Erőforrások megosztása külső partnerekkel való**. Külső üzleti partnerek ad hozzá a meglévő identitáskezelési megoldások nehézség megakadályozhatja, hogy az erőforrások hatékony megosztási és az üzleti kommunikáció.
- **A helyszíni identitás-függőségek**. Az örökölt hitelesítési mechanizmusok vagy harmadik felek többtényezős hitelesítés nem feltétlenül érhető el a felhőben, retooled, vagy áttelepítése számítási feladatokat igénylő és további identitás szolgáltatások a felhőben üzembe helyezni. Egyik követelménynek sikerült késleltetés vagy áttelepítés megakadályozza, és növelheti a költségeket.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-üzleti kockázatok, amelyek valószínűleg vezeti be a jelenlegi cloud bevezetési tervet.

Ha valós üzleti kockázatai megismerése létrejött, a következő lépés az dokumentálni az üzleti szintű kockázatokat és a mutatók és a kulcs metrikákat kíván monitorozni a tolerancia.

> [!div class="nextstepaction"]
> [Megismerheti a mutatók, metrikákat és kockázattűrés](./metrics-tolerance.md)