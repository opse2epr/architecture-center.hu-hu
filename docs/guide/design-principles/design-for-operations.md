---
title: Műveletek kialakítása
description: Az alkalmazás tervezési úgy, hogy a műveleti csapata azokat az eszközöket
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 76338cc27daf82ccb99df4e4c51c7a5ac6f26065
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="design-for-operations"></a>Műveletek kialakítása

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Az alkalmazás tervezési úgy, hogy a műveleti csapata azokat az eszközöket

A felhő jelentősen megváltozott a műveleti csapata szerepe. Már nem azok a hardver és az alkalmazást futtató infrastruktúra felelős.  Említett, műveletek része továbbra is kritikus sikeres felhő alkalmazást futtat. A műveleti csapata, fontos funkciók a következők:

- Környezet
- Figyelés
- Eszkalációs
- Incidensmegoldás
- A biztonsági naplózás

Robusztus naplózás és nyomkövetés különösen fontos az olyan felhőalapú alkalmazásokhoz. Tartalmaz, amely a műveletek csoporthoz tervezése és kialakítása, az alkalmazás biztosítja azokat az adatokat, és betekintést thay kell ahhoz, hogy sikeresen biztosításához.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Javaslatok

**Ellenőrizze az összes dolgot megfigyelhető**. Ha a megoldás központilag telepítve és fut, naplófájlok és nyomkövetési a rendszer az elsődleges betekintést. *Nyomkövetés* rögzíti a rendszer egy elérési úttal, és hasznos a szűk keresztmetszetek, a teljesítményproblémák és a hiba pontok beazonosításában. *Naplózás* például az alkalmazás állapotának megváltozásakor, hibákat és kivételeket az egyes eseményeket rögzíti. Jelentkezzen be az üzemi, ellenkező esetben ha esetleg szükség lenne rá a legtöbb nagyon időpontokban insight elvesznek.

**Figyelés eszköz**. Figyelési milyen jól (vagy nem megfelelően) az alkalmazás működik-e, rendelkezésre állási, a teljesítmény és a rendszer állapotának tekintetében betekintést biztosít. Például figyelés megtudhatja, hogy vannak teljesíti az SLA-t. Figyelési történik, a rendszer a normál működés során. Kell időhöz lehetséges, hogy a műveleti személyzet reagálhasson problémák gyorsan. Ideális esetben figyelési segít elhárítani problémák ahhoz, azok kritikus hibát okozhat. További információkért lásd: [megfigyelési és diagnosztikai][monitoring].

**Eszköz kiváltó okának elemzése**. Legfelső szintű okát az a folyamat, hogy a rendszer az alapul szolgáló hibák okait. Már elvégzett hiba után következik be. 

**Használjon elosztott nyomkövetési**. Ezzel egy elosztott nyomkövetési rendszer párhuzamossági asynchrony és felhőbeli skálázással. Nyomok tartalmaznia kell egy szolgáltatás határain túlnyúló forgalomáramlás korrelációs azonosító. Egy művelettel több alkalmazás szolgáltatáshoz intézett hívások járhat. Ha egy művelet meghiúsul, a korrelációs azonosító segít a rögzítési ponthoz a hiba oka. 

**Naplók és a metrikák szabványosítása**. A műveleti csapata származó összesített naplók kell között a különböző szolgáltatásokhoz a megoldásban. Minden szolgáltatás a saját naplózási formátumot használja, ha válik nehéz vagy lehetetlen hasznos információk lekérése. Adja meg a közös séma, például a korrelációs azonosító, az esemény nevét, IP-címet a feladó mezőket tartalmazó, és így tovább. Egyes szolgáltatások származtatni a alap séma öröklő egyéni sémák, és további mezőket tartalmaz.

**Felügyeleti feladatok automatizálására**, beleértve a kiépítés, telepítési és figyelési. A feladatok automatizálása révén megismételhető és kevesebb a hibalehetőség emberi. 

**Konfigurációs tekinti kód**. Ellenőrizze egy verziókezelő rendszer, a konfigurációs fájlok, így nyomon követheti és verzió a módosításokat, és állítsa vissza, ha szükséges. 


<!-- links -->

[monitoring]: ../../best-practices/monitoring.md


