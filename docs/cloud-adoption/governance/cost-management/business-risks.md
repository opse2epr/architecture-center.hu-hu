---
title: 'CAF: A Cost Management motivációit és az üzleti kockázatok'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A Cost Management motivációit és az üzleti kockázatok
author: BrianBlanchard
ms.openlocfilehash: b9bf31a796ea1a7530c6a668a231d74b9b765509
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299387"
---
# <a name="cost-management-motivations-and-business-risks"></a>A Cost Management motivációit és az üzleti kockázatok

Ez a cikk ismerteti az okokat, hogy ügyfeleink a Cost Management területen belül a felhő adatirányítási stratégia általában fogad el. Emellett biztosítja az üzleti kockázatok néhány példa adott meghajtó házirend-utasítások.

<!-- markdownlint-disable MD026 -->

## <a name="is-cost-management-relevant"></a>Fontos Cost Management?

Irányítási költség szempontjából a felhőre való áttérés egy paradigmát shift hoz létre. A hagyományos helyszíni világában költség felügyeleti adatfrissítési ciklusok, datacenter beszerzések, gazdagép megújításokat és ismétlődő karbantartási problémák alapul. Előrejelzés, megtervezése, és ezeket a díjakat éves tőkeráfordítási költségvetések igazodva mindegyike pontosításához.

Felhőalapú megoldásokhoz számos vállalat általában a Cost Management egy több reaktív megközelítés igénybe vehet. Sok esetben lesz prepurchase, vagy szeretné használni, a cloud services időkereten véglegesítése. Ez a modell azt feltételezi, hogy jelentős kedvezményeket, mekkora az üzleti tervek a költségkeret-beállítási egy adott felhőbeli gyártó alapján hoz létre a proaktív, tervezett költség ciklus érzete. Azonban, hogy érzete csak válik valóra, ha az üzleti is megvalósul az érett Cost Management szakterületen.

A felhőbeli önkiszolgáló képességek, amelyek korábban átláthatóságról kívüli a hagyományos helyszíni adatközpontokhoz kínál. Ezek az új képességek lehetővé teheti a vállalatok hatékonyabb, kevésbé korlátozó, és több nyitott új technológiák bevezetéséhez. A hátránya az önkiszolgáló viszont, hogy a végfelhasználók tudtukon túlléphetik lefoglalt költségvetése. Ezzel szemben ugyanazokat a felhasználókat változást észlel a tervek és váratlanul használja a cloud services előrejelzett mennyisége. A SHIFT billentyűt bármelyik irányba lehetséges indokolja, befektetés a Cost Management fegyelem a cégirányítási csapaton belüli.

## <a name="business-risk"></a>Üzleti kockázat

A Cost Management fegyelem próbál meg címet alapvető üzleti kockázatai kapcsolatos számítási feladatok felhőbeli üzemeltetése esetén felmerülő költségek. Az üzleti kockázatok azonosítása, és azok a relevancia alapján végzett figyelésére, tervezése és megvalósítása a felhőalapú környezetekben dolgozhat.

Kockázatok közös költség kapcsolatos kockázatokat használható kiindulási pontként vitafórumok a felhő Cégirányítási csapaton belül, szervezet, de a következő szolgáltatása között eltérőek:

- **Költségvetés-ellenőrzés**. Nem elsődlegesek a költségvetés vezethet a túlzott kiadások egy felhőbeli gyártójával.
- **Kihasználtság adatveszteség**. Prepurchases vagy a nem használt precommitments feleslegesen, befektetés eredményezhet.
- **Rendellenességek kiadások**: Váratlan megnövekedésére bármelyik irányba lehet a nem megfelelő használati mutatók.
- **Szükségesnél több erőforrással ellátott adategységek**. Ha az eszközök, amelyek túllépik az igényeinek, alkalmazás vagy virtuális gép (VM) konfigurációban telepített, növelheti a költségeket, és veszteség létrehozása.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-üzleti kockázatok, amelyek valószínűleg vezeti be a jelenlegi cloud bevezetési tervet.

Ha valós üzleti kockázatai megismerése létrejött, a következő lépés az dokumentálni az üzleti szintű kockázatokat és a mutatók és a kulcs metrikákat kíván monitorozni a tolerancia.

> [!div class="nextstepaction"]
> [Megismerheti a mutatók, metrikákat és kockázattűrés](./metrics-tolerance.md)
