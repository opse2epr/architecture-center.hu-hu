---
title: Tervezzen horizontális felskálázásra
description: A felhőalapú alkalmazásokat a horizontális skálázást szem előtt tartva kell megtervezni.
author: MikeWasson
ms.openlocfilehash: 8207f322d4312f6a30a8b0db7328b272c1d82de0
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206943"
---
# <a name="design-to-scale-out"></a>Tervezzen horizontális felskálázásra

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Tervezze úgy az alkalmazását, hogy az horizontálisan skálázható legyen

A felhő egyik legfőbb előnye a rugalmas méretezés &mdash; ez a kapacitás igény szerinti kihasználását, vagyis a terhelés mértéke szerinti horizontális fel- illetve leskálázást jelenti. Tervezze úgy az alkalmazását, hogy az skálázható legyen horizontálisan, igény szerint hozzá lehessen adni vagy el lehessen távolítani új példányokat.

## <a name="recommendations"></a>Ajánlatok

**Példány tartós használatának elkerülése**. A tartós használat vagy *munkamenet-affinitás* azt a gyakorlatot jelenti, amikor az ugyanazon ügyféltől érkező kérelmeket a rendszer mindig ugyanarra a kiszolgálóra irányítja. A tartós használat korlátozza az alkalmazás horizontális felskálázási képességét. Például a nagy forgalmú felhasználóktól érkező terhelés nem lesz elosztva a példányok között. A tartós használat oka lehet a memóriában tárolt munkamenet-állapot vagy a számítógép-specifikus titkosítási kulcsok használata. Gondoskodjon arról, hogy a példányok bármilyen kérelmet képesek legyenek kezelni. 

**Szűk keresztmetszetek azonosítása**. A horizontális felskálázással nem lehet varázsütésre megoldani az összes teljesítménybeli problémát. Ha például a háttéradatbázis a szűk keresztmetszet, a további webkiszolgálók hozzáadása nem sokat segít. Még több példány bevetése helyett a szűk keresztmetszetek azonosításával és feloldásával kell kezdeni a probléma megoldását. A szűk keresztmetszeteket általában a rendszer állapottal rendelkező összetevői okozzák. 

**Számítási feladatok lebontása a méretezhetőségi követelmények szerint.**  Az alkalmazások gyakran több számítási feladatból állnak, amelyek eltérő méretezési követelményekkel rendelkeznek. Például egy alkalmazásnak lehet egy nyilvános helye, valamint egy különálló felügyeleti helye. A nyilvános hely esetében előfordulhat hirtelen forgalomnövekedés, míg a felügyeleti hely kisebb, kiszámíthatóbb terheléssel dolgozik. 

**Erőforrás-igényes feladatok kiszervezése.** Ha lehetséges, a nagy processzorteljesítményt vagy I/O-erőforrásokat igénylő feladatokat át kell helyezni [a háttérfeladatokhoz][background-jobs], hogy a felhasználói kérelmek kezelését végző előtérrendszer terhelése minimalizálható legyen.

**Beépített automatikus skálázási szolgáltatások használata**. Sok Azure számítási szolgáltatás tartalmaz beépített támogatást az automatikus skálázásra vonatkozóan. Ha az alkalmazás kiszámítható és rendszeres számítási feladatokat lát el, a horizontális felskálázás történhet ütemezés szerint. Például a horizontális felskálázás végrehajtható munkaidőben. Ha azonban a számítási feladatok nem jelezhetők előre, akkor teljesítménymutatókat, például a processzor vagy a kérelem üzenetsorának hosszát érdemes alkalmazni az automatikus skálázás aktiválásához. A automatikus skálázáshoz kapcsolódó ajánlott eljárásokat lásd: [Automatikus skálázás][autoscaling].

**A kritikus számítási feladatok esetében érdemes megfontolni az agresszív automatikus skálázást**. A kritikus számítási feladatok esetében jobb egy lépéssel az igények előtt járni. Érdemesebb a nagy terhelés alatt gyorsan új példányokat hozzáadni a megnövekedett forgalom kezelésére, majd fokozatosan csökkenteni az erőforrások számát.

**Tervezzen a horizontális leskálázást szem előtt tartva**.  Nem szabad elfelejteni, hogy a rugalmas skálázás mellett az alkalmazás esetében időnként sor fog kerülni a horizontális leskálázásra, amikor a példányok törlődnek. Az alkalmazásnak megfelelően kell kezelnie a példányok eltávolítását. Íme néhány példa a horizontális leskálázás kezelésére:

- Figyelje a leállítási eseményeket (ha ez elérhető), és biztosítsa, hogy a leállítás szabályszerűen történjen. 
- Egy szolgáltatás ügyfeleinek/felhasználóinak támogatniuk kell az átmeneti hibák kezelését és az újrapróbálkozást. 
- Hosszan futó feladatok esetében érdemes felosztani a munkát, ellenőrzőpontok vagy a [Csövek és szűrők][pipes-filters-pattern] mintájának használatával. 
- Helyezze a munkaelemeket egy üzenetsorba, így ha egy példány a feldolgozás közepén kiesik, egy másik folytathatja a munkát. 


<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
