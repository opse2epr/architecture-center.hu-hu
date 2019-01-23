---
title: Leépítő minta
titleSuffix: Cloud Design Patterns
description: Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f139d368c98256c0190753930983a47df81a5134
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480558"
---
# <a name="strangler-pattern"></a>Leépítő minta

Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél. A korábbi rendszer funkcióinak lecserélésével idővel az új rendszer felváltja a régi rendszer összes funkcióját, leépítve a régi rendszert, és lehetővé téve annak leszerelését.

## <a name="context-and-problem"></a>Kontextus és probléma

A rendszerek öregedésével az azok alapjául szolgáló fejlesztési eszközök, üzemeltetési technológia és a rendszerarchitektúrák is jelentősen elavulttá válhatnak. Új szolgáltatások és funkciók hozzáadásával ezen alkalmazások összetettsége jelentősen nőhet, így a karbantartásuk, illetve a további funkciók hozzáadása nehezebb lesz.

Egy összetett rendszer teljes cseréje hatalmas feladat lehet. Gyakran fokozatosan kell áttérni az új rendszerre a régi rendszer megtartása mellett a még nem migrált funkciók kezelése céljából. Egy alkalmazás két különálló verziójának futtatásakor azonban az ügyfeleknek tudniuk kell, hol találhatóak az egyes funkciók. Funkciók vagy szolgáltatások migrálásakor az ügyfeleket minden alkalommal frissíteni kell, hogy az új helyre mutassanak.

## <a name="solution"></a>Megoldás

Fokozatosan cserélje le az egyes funkciókat új alkalmazásokra és szolgáltatásokra. Egy előtérrendszer létrehozásával elfoghatja a korábbi háttérrendszernek címzett kérelmeket. Az előtérrendszer ezeket a kérelmeket a korábbi alkalmazáshoz vagy az új szolgáltatásokhoz irányítja. A meglévő szolgáltatások fokozatosan migrálhatóak az új rendszerre, és a fogyasztók továbbra is ugyanazt a felületet használhatják anélkül, hogy bármit érzékelnének a migrálásból.

![A Leépítő minta ábrája](./_images/strangler.png)

Ez a minta segít minimálisra csökkenteni a migrálással járó kockázatokat, és időben jobban elosztja a fejlesztési folyamatot. Miközben az előtérrendszer biztonságosan a megfelelő alkalmazáshoz irányítja a felhasználókat, tetszés szerint ütemezve adhat funkciókat az új rendszerhez a korábbi alkalmazás működésének fenntartása mellett. A funkciók új rendszerre való migrálása idővel „leépíti” a régi rendszer funkcionalitását, és nem lesz rá többet szükség. A folyamat befejeződése után a régi rendszer biztonságosan megszüntethető.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Fontolja meg, hogyan kívánja kezelni azokat a szolgáltatásokat és adattárakat, amelyeket esetlegesen az új és a régi rendszer is használhat. Biztosítsa, hogy a két rendszer egyidejűleg hozzáférhessen ezekhez az erőforrásokhoz.
- Úgy alakítsa ki az új alkalmazások és szolgáltatások szerkezetét, hogy könnyen elfoghatóak és lecserélhetőek legyenek a jövőbeli leépítési migrálások során.
- A migrálás befejeződése után a leépítési előtérrendszer megszüntethető, vagy régebbi ügyfelek adapterévé alakítható.
- Gondoskodjon róla, hogy az előtérrendszer lépést tartson a migrálással.
- Ügyeljen rá, hogy az előtérrendszer ne váljon kritikus meghibásodási ponttá, és ne képezhessen teljesítménybeli szűk keresztmetszetet.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ezt a mintát háttéralkalmazások új architektúrákra való fokozatos migrálásához használhatja.

Nem érdemes ezt a mintát használni, ha:

- A háttérrendszernek küldött kérelmek nem foghatók el.
- Kisebb rendszerek esetében, ahol a teljes körű lecserélés összetettsége alacsony.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- A Martin Fowler blogbejegyzés [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)
