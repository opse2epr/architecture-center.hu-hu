---
title: Sérülésgátló réteg minta
description: Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ac898519c9aa0a0aa2301da9f48756db0eb2af7c
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
---
# <a name="anti-corruption-layer-pattern"></a>Sérülésgátló réteg minta

Egy homlokzat vagy adapter réteg között különböző alrendszereket, ne ossza meg az azonos szemantikáját megvalósításához. Ez a réteg az eszköz egy alrendszer hajt végre a más alrendszer kérelmek. Ez a minta segítségével győződjön meg arról, hogy egy alkalmazás tervét nem korlátozza a külső alrendszereket függőségek. A mintáról először Eric Evans írt *Domain-Driven Design* című könyvében.

## <a name="context-and-problem"></a>Kontextus és probléma

A legtöbb alkalmazás más rendszerekre támaszkodik bizonyos adatokért vagy funkciókért. Ha például egy örökölt alkalmazást egy modern rendszerbe migrálnak, annak továbbra is szüksége lehet a meglévő régebbi erőforrásokra. Az új funkcióknak képesnek kell lenniük meghívni a régi rendszert. Ez különösen igaz fokozatos migrálás esetén, ahol az idő előrehaladtával egy nagyobb alkalmazás különböző funkcióit helyezik át egy modern rendszerbe.

A régebbi rendszerekben gyakoriak a minőségbeli hibák (pl. szövevényes adatsémák, elavult API-k). A korábbi rendszerek funkciói és technológiái nagyban eltérhetnek a modern rendszerekétől. Ahhoz, hogy az új alkalmazás együttműködhessen a régebbi alkalmazással, nagy eséllyel támogatnia kell elavult infrastruktúrákat, protokollokat, adatmodelleket, API-kat vagy más olyan funkciókat, amelyeket máskülönben nem tennénk egy korszerű alkalmazás részévé.

Az új és a régebbi rendszer közötti hozzáférés fenntartása érdekében előfordulhat, hogy az új rendszernek alkalmazkodnia kell néhányhoz a korábbi rendszer API-jai vagy más szemantikái közül. Ha ezek a régebbi funkciók hibásak, támogatásukkal „megsérül” a máskülönben letisztult tervezésű korszerű alkalmazás. 

Bármely külső rendszer szabályozó a fejlesztői csapat nem, csak a régebbi rendszerekre hasonló problémák merülhetnek fel. 

## <a name="solution"></a>Megoldás

Különítse el a különböző alrendszereket úgy, hogy egy víruskereső sérülés réteg között. Ez a réteg lefordítja a két rendszer, így egy rendszer változatlanok maradnak, amíg a másik elkerülése érdekében a tervezési és technológiai megközelítést veszélyeztetése közötti kommunikáció.

![](./_images/anti-corruption-layer.png) 

A fenti ábra mutatja az alkalmazás két alrendszereket. Alrendszer A hívások alrendszer B keresztül egy víruskereső sérülés réteget. Alrendszer közötti kommunikációt és a sérülés elleni réteg mindig használja az adatmodellben és az architektúra alrendszer A. hívások a víruskereső sérülés rétegből alrendszer B felelnek meg, hogy alrendszer adatmodell vagy módszert. A sérülésgátló réteg tartalmazza az összes logikát, amely a két rendszer közötti fordításhoz szükséges. A réteget implementálhatja egy alkalmazáson belüli összetevőként, de akár független szolgáltatásként is.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- A sérülésgátló réteg késéseket okozhat a két rendszer közötti hívásoknál.
- A sérülésgátló réteg egy további szolgáltatást képez, amely kezelést és karbantartást igényel.
- Gondolja át, hogyan lesz skálázva a sérülésgátló réteg.
- Fontolja meg, hogy egynél több sérülésgátló rétegre lenne-e szüksége. Érdemes lehet funkció szerint felbontani több, különböző technológiákat és nyelveket használó szolgáltatásra, de más okokból is particionálhatja a sérülésgátló réteget.
- Gondolja át, hogy egyéb alkalmazásaihoz vagy szolgáltatásaihoz képest hogyan fogja kezelni a sérülésgátló réteget. Hogyan fogja integrálni a monitorozási, kiadási és konfigurációs folyamataiba?
- Győződjön meg arról, hogy a tranzakciós és adatkonzisztencia fenntartható és monitorozható.
- Vegye figyelembe, hogy kell-e a víruskereső sérülés réteg különböző alrendszereket, vagy a szolgáltatások egy részhalmaza közötti összes kommunikáció kezelésére. 
- Ha a víruskereső sérülés réteg az alkalmazás áttelepítési stratégiájának a részét, fontolja meg a végleges, vagy rendszerből összes örökölt funkció migrálása befejeződött.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Több lépésben tervez migrációt végrehajtani, de szükség van a régi és az új rendszer közötti integráció fenntartására.
- Két vagy több alrendszereket különböző szemantikáját, de továbbra is kell való kommunikációhoz. 

Nem érdemes ezt a mintát használni, ha nincsenek jelentős szemantikai különbségek az új és a régebbi rendszer között. 

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Strangler minta](./strangler.md)
