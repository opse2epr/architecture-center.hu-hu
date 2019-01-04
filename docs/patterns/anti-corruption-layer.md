---
title: Sérülésgátló réteg minta
titleSuffix: Cloud Design Patterns
description: Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: d1023140deea4a2714c762945935d0838136e508
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011276"
---
# <a name="anti-corruption-layer-pattern"></a>Sérülésgátló réteg minta

Egy előtérrendszer vagy implementálhat között különböző alrendszereket, amelyek nem azonos. Ez a réteg lefordítja a kéréseket, amelyek a többi alrendszer egy alrendszer hajt végre. Ez a minta segítségével győződjön meg arról, hogy egy alkalmazás-tervezés nem korlátozódik alrendszereket külső függőségei. A mintáról először Eric Evans írt *Domain-Driven Design* című könyvében.

## <a name="context-and-problem"></a>Kontextus és probléma

A legtöbb alkalmazás más rendszerekre támaszkodik bizonyos adatokért vagy funkciókért. Ha például egy örökölt alkalmazást egy modern rendszerbe migrálnak, annak továbbra is szüksége lehet a meglévő régebbi erőforrásokra. Az új funkcióknak képesnek kell lenniük meghívni a régi rendszert. Ez különösen igaz fokozatos migrálás esetén, ahol az idő előrehaladtával egy nagyobb alkalmazás különböző funkcióit helyezik át egy modern rendszerbe.

A régebbi rendszerekben gyakoriak a minőségbeli hibák (pl. szövevényes adatsémák, elavult API-k). A korábbi rendszerek funkciói és technológiái nagyban eltérhetnek a modern rendszerekétől. Ahhoz, hogy az új alkalmazás együttműködhessen a régebbi alkalmazással, nagy eséllyel támogatnia kell elavult infrastruktúrákat, protokollokat, adatmodelleket, API-kat vagy más olyan funkciókat, amelyeket máskülönben nem tennénk egy korszerű alkalmazás részévé.

Az új és a régebbi rendszer közötti hozzáférés fenntartása érdekében előfordulhat, hogy az új rendszernek alkalmazkodnia kell néhányhoz a korábbi rendszer API-jai vagy más szemantikái közül. Ha ezek a régebbi funkciók hibásak, támogatásukkal „megsérül” a máskülönben letisztult tervezésű korszerű alkalmazás.

Bármilyen külső rendszerrel, a fejlesztői csapat nem szabályozhatja, csak a régebbi rendszerekkel hasonló problémák merülhetnek fel.

## <a name="solution"></a>Megoldás

Ezek között egy sérülésgátló réteg elhelyezésével elkülönítheti a különböző alrendszereket. Ez a réteg lefordítja a két rendszer, így több rendszer változatlan maradhat a másik elkerülheti a saját tervezése és technológiai megközelítése veszélyeztetése közötti kommunikációt.

![A Sérülésgátló réteg minta ábrája](./_images/anti-corruption-layer.png)

A fenti ábrán egy két alrendszerek alkalmazás látható. Egy alrendszer meghívja a B alrendszer egy sérülésgátló réteg keresztül. Alrendszer közötti kommunikációt és a sérülésgátló réteg mindig használja az adatmodellben és A. hívásait a sérülésgátló réteg alrendszerhez B felelnek meg adott alrendszer adatmodelljéhez vagy metódusaihoz alrendszer architektúráját. A sérülésgátló réteg tartalmazza az összes logikát, amely a két rendszer közötti fordításhoz szükséges. A réteget implementálhatja egy alkalmazáson belüli összetevőként, de akár független szolgáltatásként is.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- A sérülésgátló réteg késéseket okozhat a két rendszer közötti hívásoknál.
- A sérülésgátló réteg egy további szolgáltatást képez, amely kezelést és karbantartást igényel.
- Gondolja át, hogyan lesz skálázva a sérülésgátló réteg.
- Fontolja meg, hogy egynél több sérülésgátló rétegre lenne-e szüksége. Érdemes lehet funkció szerint felbontani több, különböző technológiákat és nyelveket használó szolgáltatásra, de más okokból is particionálhatja a sérülésgátló réteget.
- Gondolja át, hogy egyéb alkalmazásaihoz vagy szolgáltatásaihoz képest hogyan fogja kezelni a sérülésgátló réteget. Hogyan fogja integrálni a monitorozási, kiadási és konfigurációs folyamataiba?
- Győződjön meg arról, hogy a tranzakciós és adatkonzisztencia fenntartható és monitorozható.
- Vegye figyelembe, hogy a sérülésgátló réteg különböző alrendszereket, vagy a szolgáltatások egy részhalmaza közötti összes kommunikációt kezelnie kell-e.
- Ha a sérülésgátló réteg egy alkalmazás áttelepítési stratégiájának a részét, érdemes e állandó lesz, vagy kivonjuk a forgalomból Elvégre örökölt funkciót migrált.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Több lépésben tervez migrációt végrehajtani, de szükség van a régi és az új rendszer közötti integráció fenntartására.
- Két vagy több alrendszer különböző szemantikával rendelkezik, de mégis kommunikálniuk kell.

Nem érdemes ezt a mintát használni, ha nincsenek jelentős szemantikai különbségek az új és a régebbi rendszer között.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Leépítő minta](./strangler.md)
