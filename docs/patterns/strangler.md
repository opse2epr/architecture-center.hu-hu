---
title: Strangler minta
description: Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: d03e8a1ef9077b6e00ea5a17423bf7e09b68111a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540985"
---
# <a name="strangler-pattern"></a>Strangler minta

Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal. Lecseréli a korábbi rendszer szolgáltatásait, mert az új rendszer végül lecseréli a régi rendszerre a Funkciók, a régi rendszerre strangling, és lehetővé téve az leszerelése. 

## <a name="context-and-problem"></a>A környezetben, és probléma

Mivel volt épülő rendszerek élettartamát a fejlesztői eszközök, üzemeltetési technológia, és még akkor is, függvényében egyre válhat elavult. Új szolgáltatások és funkciók hozzáadása, ezek az alkalmazások összetettsége jelentősen növekedhet, így nehezebb karbantartása, vagy adja hozzá az új szolgáltatások.

Egy összetett rendszer teljes csere hatalmas vállalkozás is lehet. Gyakran szüksége lesz egy fokozatos áttelepítés új rendszerre, miközben megőrzi a régi rendszerre funkciókat, amelyek még nem lettek áttelepítve kezelésére. Azonban az alkalmazás két külön verzióit futtató azt jelenti, hogy az ügyfelek tudni, hogy hol találhatók az adott szolgáltatások. Minden alkalommal, amikor egy szolgáltatás vagy szolgáltatás áttelepítése, ügyfelek frissítenie kell a helyre mutasson.

## <a name="solution"></a>Megoldás

Növekményesen cserélje funkció nehezen új alkalmazásokhoz és szolgáltatásokhoz. Hozzon létre egy homlokzati, amely elfogja a beérkező kéréseket a háttérrendszer örökölt címen. A homlokzati ezeket a kérelmeket, vagy az örökölt alkalmazást vagy az új szolgáltatások irányítja. Meglévő szolgáltatások telepíthetők át az új rendszerre fokozatosan, és a felhasználók továbbra is használhatják az ugyanazon a felületen érzékelte, hogy az áttelepítési megtörtént.

![](./_images/strangler.png)  

Ez a minta segítségével az áttelepítésből minimálisra, és fejlesztési részéről az erőfeszítés eloszlatva idő. Biztonságosan útválasztási a felhasználók a megfelelő alkalmazás homlokzati hozzáadhat funkció az új rendszerre bármilyen ütemben van lehetősége, miközben gondoskodik az örökölt alkalmazás továbbra is működik. Idővel szolgáltatások áttelepítése az új rendszerre, a korábbi rendszer van végül "strangled" és már nem szükséges. Ez a folyamat befejeződése után a korábbi rendszer biztonságosan lehet visszavonni.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Fontolja meg az új és a korábbi rendszerek által esetleg használt szolgáltatásaikat és adataikat tárolók kezelése. Győződjön meg arról, hogy mindkét hozzáférhet az ezen erőforrások-mellé.
- Struktúra új alkalmazások és szolgáltatások oly módon, hogy azok könnyen megszerezhetik, és lecseréli a jövőben strangler áttelepítéseket.
- Bármikor az áttelepítés akkor fejeződött be, amikor a strangler homlokzati fog eltűnik vagy be egy adapter esetében a hagyományos fejleszteni.
- Ellenőrizze, hogy a homlokzati összhangban van-e az áttelepítés.
- Ellenőrizze, hogy a homlokzati egyetlen pont, hibát vagy a teljesítménybeli szűk keresztmetszetek nem válik.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ezt a mintát használja, egy háttér-alkalmazás egy új architektúra fokozatosan áttelepítésekor.

Ez a minta nem alkalmasak lehetnek:

- Ha a kérelmeket a háttér-rendszer elfogta nem.
- Azon kisebb rendszerekhez nagybani helyettesítő összetettsége alacsony.

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Víruskereső sérülés réteg minta](./anti-corruption-layer.md)
- [Átjáró útválasztás minta](./gateway-routing.md)


 

