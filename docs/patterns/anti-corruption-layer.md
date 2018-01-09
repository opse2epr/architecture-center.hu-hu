---
title: "Víruskereső sérülés réteg minta"
description: "Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: e41f080abbef772596ee7f8b10ad72bb03a3b829
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/05/2018
---
# <a name="anti-corruption-layer-pattern"></a>Víruskereső sérülés réteg minta

A modern alkalmazás- és egy örökölt, amelyektől függ közötti homlokzat vagy adapter rétegben megvalósításához. Ez a réteg az eszköz közötti a modern alkalmazás- és az örökölt kérelmek. Ez a minta segítségével győződjön meg arról, hogy a régebbi rendszerekre függőségek nem korlátozza az alkalmazás tervét. Ebben a mintában először szerint Eric Evans a *Domain-Driven tervezési*.

## <a name="context-and-problem"></a>A környezetben, és probléma

A legtöbb alkalmazás bizonyos adatokhoz vagy funkciókhoz más rendszerek támaszkodnak. Például ha egy örökölt alkalmazás modern rendszerre telepít át, az továbbra is esetleg meglévő hagyományos erőforrásokat. Új szolgáltatások korábbi rendszer képesnek kell lennie. Ez különösen igaz a fokozatos áttelepítéseket, ahol egy nagyobb alkalmazás különböző funkcióit áthelyezik egy modern rendszer adott idő alatt.

A régebbi rendszerekre gyakran érinti a minőségi problémákat, például csavart adatok sémák vagy elavult API-t. A szolgáltatások és korábbi rendszerekben használt technológiák széles körben változhat korszerűbb rendszerek. A korábbi rendszerrel működik, az új alkalmazás támogatja az elavult infrastruktúra, protokollok, adatmodellekben, API-k vagy egyéb szolgáltatásokat, amelyek a modern alkalmazások egyébként nem tudnák üzembe kell.

Az új rendszer igazodnia kell legalább egy új és korábbi rendszerek közötti hozzáférés fenntartásához kényszerítheti a korábbi rendszer API-k vagy más szemantikáját. Ha a régi szolgáltatások minőségének problémák vannak, azokat támogató "megsérülnek" milyen más módon lehet egy szabályszerűen tervezett modern alkalmazások. 

## <a name="solution"></a>Megoldás

Különítse el a régebbi és modern rendszerek úgy, hogy egy víruskereső sérülés réteg között. Ez a réteg lefordítja a két rendszer, így a korábbi rendszer változatlanok maradnak, amíg a modern alkalmazásnak a tervezési és technológiai megközelítést veszélyeztetése elkerülése érdekében közötti kommunikáció.

![](./_images/anti-corruption-layer.png) 

A modern alkalmazások és a sérülés elleni réteg közötti kommunikáció mindig az alkalmazás adatokat az adatmodellbe és architektúra használja. Az örökölt rendszer hívásainak a víruskereső sérülés réteg által adott rendszer adatmodell vagy módszerek felelnek meg. A víruskereső sérülés réteg tartalmazza az összes szükséges lefordítani a két rendszer között logika. A réteg összetevőjeként az alkalmazáson belül, vagy egy független szolgáltatásként valósítható meg.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- A víruskereső sérülés réteg késés adhat hozzá a két rendszer hívások.
- A víruskereső sérülés réteg hozzáadja egy másik szolgáltatást, amely felügyelt és karbantartani kell.
- Vegye figyelembe, hogyan lesz skálázva a víruskereső sérülés réteg.
- Vegye figyelembe, hogy szükséges-e egynél több víruskereső sérülés réteg. Érdemes lehet felbontani funkció szolgáltatásba több különböző technológiákkal, illetve azokat a nyelveket, vagy előfordulhat, hogy a víruskereső sérülés réteg particionálásához más okok miatt.
- Vegye figyelembe, hogy a víruskereső sérülés réteg hogyan kezelik a más alkalmazásokkal vagy szolgáltatásokkal kapcsolatban. Hogyan fogja azt integrálni kell a figyelés, release és konfigurációs folyamatok?
- Győződjön meg arról, tranzakció és az adatok konzisztenciájának megmaradnak, és figyelhető.
- Vegye figyelembe, hogy a víruskereső sérülés réteg kell-e a hagyományos és modern rendszerek vagy szolgáltatások egy részhalmaza közötti összes kommunikáció kezelésére. 
- Vegye figyelembe, hogy a víruskereső sérülés réteg célja, hogy végleges vagy végül kivont egyszer minden örökölt funkciója migrálása befejeződött.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Áttelepítés tervezett megtörténjen-e keresztül több szakaszt, de új és korábbi rendszerek közötti integrációs kell megőrizni.
- Új és korábbi rendszer különböző szemantikáját, de továbbra is kell való kommunikációhoz.

Előfordulhat, hogy ez a minta nem megfelelő, ha új és a korábbi rendszerek között nincs jelentős szemantikai különbségek vannak. 

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Strangler minta][strangler]

[strangler]: ./strangler.md
