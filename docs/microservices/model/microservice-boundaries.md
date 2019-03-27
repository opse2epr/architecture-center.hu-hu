---
title: Mikroszolgáltatások határainak azonosítása
description: Mikroszolgáltatások határainak azonosítása.
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 69213701b87f052811a1f032c056f05be7097c14
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299033"
---
# <a name="identifying-microservice-boundaries"></a>Mikroszolgáltatások határainak azonosítása

Mi az a megfelelő méretű, a mikroszolgáltatások? Gyakran Hallott valamit a hatását, "túl nagy és túl kicsi" &mdash; , és bár ez természetesen megfelelő, nem a gyakorlatban nagyon hasznos. De ha elindítja a gondosan megtervezett tartományi modell, sokkal egyszerűbb mikroszolgáltatás-alapú vonatkozó döntések meghozatalát.

![Kapcsolódó kontextusokat ábrája](../images/bounded-contexts.png)

Ez a cikk egy drónos szállítási szolgáltatást használ egy futó példát. Talál további információt a forgatókönyv és a megfelelő megvalósításának hivatkozhat [Itt](../design/index.md).

## <a name="from-domain-model-to-microservices"></a>A tartományi modellben mikroszolgáltatások

Az a [előző cikk](./domain-analysis.md), a Drone Delivery alkalmazás kapcsolódó kontextusokat készletét meghatározott. A kapcsolódó kontextusokat, ezek a szállítási korlátozott környezet egyik jobban megvizsgálta és azonosítja az entitásokban, az összesítések, és a tartományi szolgáltatásokat, amelyek korlátozott környezet.

Most már készen állunk nyissa meg a tartományi modellben alkalmazás-tervezés. Itt látható, amely mikroszolgáltatások származtassa a tartományi modellben használható.

1. Kezdje egy körülhatárolt kontextus. A funkció a mikroszolgáltatások általában nem kell span egynél több körülhatárolt kontextus. Definíció szerint egy körülhatárolt kontextus egy adott modell határait jelöli meg. Ha azt tapasztalja, hogy mikroszolgáltatások együtt eredményét-e ugyanaz a tartományi modellekkel, lépjen vissza és finomíthatja a tartományelemzés szükség lehet, hogy ez.

2. Ezután tekintse meg az összesítések, a tartományi modellben. Összesítések között gyakran érdemes mikroszolgáltatásokat. Egy jól megtervezett összesítés szolgáltatásnak számos egy jól megtervezett mikroszolgáltatás jellemzői, például:

    - Összesítés üzleti követelmények, nem pedig például az adatok elérése vagy az üzenetkezelési a technikai problémák származik.
    - Összesítés magas működési kohézióval kell rendelkezniük.
    - Összesítés az adatmegőrzés szegélyére.
    - Összesítések lazán kell lennie.

3. Tartományi szolgáltatások között is érdemes mikroszolgáltatásokat. Tartományi szolgáltatások állapot nélküli műveletek, amelyek között több összesítések. Egy tipikus példája egy munkafolyamatot, amely magában foglalja a több mikroszolgáltatás-alapú. Példa erre a Drone Delivery alkalmazás lesz látható.

4. Végül vegye figyelembe a nem funkcionális követelményeket. Tekintse meg a tényezők, például a csapat mérete, adattípusok, technológiák, méretezhetőségi követelményeinek, rendelkezésre állással és biztonsági követelményeket. Ezek a tényezők vezethet, hogy tovább bontható fel a mikroszolgáltatások két vagy több kisebb servicesbe, vagy ennek az ellenkezője tegye, és néhány a mikroszolgáltatások egyesítése egy.

Miután azonosította a mikroszolgáltatások az alkalmazásban, ellenőrizze a kialakítás az alábbi feltételek alapján:

- Minden szolgáltatás egyetlen felelősséget rendelkezik.
- Nincsenek forgalmas hívások nem szolgáltatások között. Ha felosztása két szolgáltatás funkciókat okoz, ha túlságosan forgalmas meg, ezek a függvények az egyazon szolgáltatáshoz tartozó szokásostól eltérő lehet.
- Egyes szolgáltatások általában elég kis méretűek, hogy azt is lehet fejlesztőcsapata által készített egy kis egymástól függetlenül működik.
- Nincsenek két vagy több szolgáltatás zárolási lépésben telepíteni fog igénylő közötti függőségek. Mindig kell egy szolgáltatás telepíthető bármilyen egyéb szolgáltatás újbóli telepítése nélkül.
- Szolgáltatások nem szorosan összekapcsolt, és egymástól függetlenül tovább lehessen fejleszteni.
- A szolgáltatások határait nem hoz létre a problémák adatkonzisztencia vagy -integritást. Egyes esetekben fontos funkciók üzembe egyetlen mikroszolgáltatást adatkonzisztencia karbantartásához. Ezzel együtt az üzemeltetés, vegye figyelembe, hogy valóban szükséges-e konzisztenciát. Végleges konzisztencia az elosztott rendszerekben címzéshez stratégiák, és a szolgáltatások gyakran decomposing előnyei nyomósabbak a végleges konzisztencia kezelése kihívást.

Mindenekelőtt fontos pragmatikus, és ne feledje, hogy tartományvezérelt tervezési iteratív folyamat. Ha kétségei vannak, indítsa el a további coarse-grained mikroszolgáltatásokat. Egyszerűbb, mint a funkció számos meglévő mikroszolgáltatások túlnyúló újrabontás mikroszolgáltatások felosztása két kisebb szolgáltatásra.
  
## <a name="example-defining-microservices-for-the-drone-delivery-application"></a>Példa: Mikroszolgáltatások a Drone Delivery alkalmazás meghatározása

Idézze, hogy a fejlesztői csapat kellett azonosítja a négy összesítések &mdash; kézbesítési, a csomag, a Drone és a fiók &mdash; és a két tartomány szolgáltatás, az ütemező és a felügyelő.

Teljesítés és a Csomagmegosztás deduplikációra nyilvánvaló mikroszolgáltatás-alapú. Az ütemező és a felügyelő koordinálja a tevékenységét más mikroszolgáltatás-alapú, ezért érdemes megvalósítani a ezeket domain servicesben mikroszolgáltatásokat.

Drónos és fiókot is érdekes, mert egyéb kapcsolódó kontextusokat tartoznak. Az egyik lehetőség az ütemező a Drone meghívásához és fiókot, amelyet környezetek közvetlenül. Egy másik lehetőség, a Drone és a fiók mikroszolgáltatások belül a szállítási korlátozott környezet létrehozásához. Ilyen mikroszolgáltatásokból lenne résidőkiosztással között a kapcsolódó kontextusokat, API-k és adatsémák, amely a szállítási címhez tartozó környezeti érdemesebb teszi elérhetővé.

A Drone, valamint a fiók, amelyet környezetekben ez az útmutató nem terjed vannak, így azok utánzatként funkcionáló szolgáltatások referenciaimplementáció létrehoztunk. De Íme néhány tényező ebben a helyzetben érdemes figyelembe venni:

- Mi az a hálózati többletterhelést okoz, közvetlenül a többi körülhatárolt kontextus-be irányuló hívás?

- Ideális a sémát a többi körülhatárolt kontextus az ebben a környezetben, vagy jobban szeretné, hogy egy sémát, amely a körülhatárolt kontextus személyre szabott?

- Az a többi körülhatárolt kontextus egy korábbi rendszer? Ha tehát, előfordulhat, hogy létrehoz egy szolgáltatást, amely indextáblaként egy [sérülésgátló réteg](../../patterns/anti-corruption-layer.md) az átalakításra a régi rendszer és a modern alkalmazás között.

- Mi az a csapat struktúra? Az egyszerű, a többi körülhatárolt kontextus felelős csapat kommunikálni? Ha nem, egy szolgáltatás, amely a két környezet közötti közvetítő létrehozása segíthet közötti kommunikáció költségeinek csökkentése érdekében.

Az eddigi azt még nem tekinthető minden olyan nem funkcionális követelmények. Az alkalmazás átviteli sebességet megkövetelő mértékegységeként, a fejlesztői csapat úgy döntött, hogy hozzon létre egy külön Adatbetöltési mikroszolgáltatás, amely az ügyfél kérelmek feldolgozására. A mikroszolgáltatások megvalósítandó [Terheléskiegyenlítés](../../patterns/queue-based-load-leveling.md) feldolgozásra a pufferbe írja a bejövő kérelmek. Az ütemező beolvassa a kérelmeket a buffer szolgáltatásból származó és a munkafolyamat végrehajtása.

Nem funkcionális követelmények a csapata egy kiegészítő szolgáltatás létrehozásához vezetett. A szolgáltatásokhoz, amennyiben az ütemezés és a csomagok továbbítása valós időben a folyamat lett. De a rendszer is kell tárolnia minden szállítás előzményei a data-elemzéshez hosszú távú tároláshoz. A csapata tekintett feladata a Licenctovábbítási szolgáltatása, ami. De az adattárolási követelmények is igen különböző az előzményadatokat és átvitel közben az operations (lásd: [adatok szempontok](../design/data-considerations.md)). Ezért a csapat úgy döntött, hogy hozzon létre egy külön szállítás előzményei szolgáltatás, amely DeliveryTracking események kézbesítés a szolgáltatás figyelésére, és írja az események, hosszú távú tároláshoz.

Az alábbi ábrán látható ezen a ponton a terv:

![Tervezési diagramja](../images/drone-delivery.png)

## <a name="next-steps"></a>További lépések

Ezen a ponton kell erre a célra egyértelművé, illetve a funkciók köre az egyes mikroszolgáltatások a tervezés. A rendszer most már Ön is tervezhet.

> [!div class="nextstepaction"]
> [Mikroszolgáltatási architektúra tervezése](../design/index.md)

<!-- links -->

[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
