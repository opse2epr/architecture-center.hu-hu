---
title: 'CAF: Üzembe helyezés gyorsítás mintautasítások házirend'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Üzembe helyezés gyorsítás mintautasítások házirend
author: alexbuckgit
ms.openlocfilehash: 4f7d59e6653c29db03f966d1c7105524b72586fc
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899295"
---
# <a name="deployment-acceleration-sample-policy-statements"></a>Üzembe helyezés gyorsítás mintautasítások házirend

Egyes felhőszolgáltatási házirend-utasítások szolgálnak, amelyeket a kockázati értékelés során azonosított adott kockázatkezelés. Ezek az utasítások kell biztosítania a kockázatok tömör összegzését, és azokkal a tervek. Minden egyes utasítás definíciójának tartalmaznia kell adatot megtalálhatja:

- **Technikai kockázatot.** Ez a szabályzat foglalkozni fog a kockázat összegzését.
- **Házirendutasítás.** A házirend követelményeinek törölje összefoglaló leírását.
- **Kialakítási lehetőségeket.** Végrehajtható javaslatok, előírásoknak vagy egyéb útmutatást, amely az IT-csoportoknak és fejlesztők is használhatják, ha a házirend megvalósítása.

A következő házirend mintautasítások cím számos gyakori üzleti konfigurációhoz kapcsolódó kockázatokat, és példaként szolgálnak, hogy a házirend-utasítások használatával saját szervezete igények kidolgozásakor hivatkozhat. Vegye figyelembe, hogy ezek a példák nem jelentenek a proscriptive kell, és a vélhetően több szabályzat lehetőség kezelésekor bármely adott kockázattal rendelkező. Szorosan együttműködnek az üzleti és informatikai csapata hozza létre, azonosíthatja a házirend a kockázatok egyedi készletét leginkább megfelelő megoldásokra.

## <a name="reliance-on-manual-deployment-or-configuration-of-systems"></a>Manuális telepítés vagy a konfiguráció rendszerek igényel

**Technikai kockázati**: Központi telepítése vagy konfigurálása során emberi beavatkozás a függő emberi hibák valószínűségét növeli és csökkenti az ismételhetőség és előreláthatóságának növelését rendszer központi telepítése és konfigurálása. Azt is általában lassabb központi telepítés rendszer vezet.

**Házirendutasítás**: Az összes eszköz üzembe helyezhető a felhőben kell telepíteni, a sablonok vagy az automatizálási szkriptek, amikor csak lehetséges.

**A lehetséges tervezési lehetőségek**: [Az Azure Resource Manager-sablonok](/azure/azure-resource-manager/resource-group-overview#template-deployment) üzembe helyezni az erőforrásokat az Azure-infrastruktúra mint kód módszert biztosít. A [Azure-építőelemekkel](https://github.com/mspnp/template-building-blocks/wiki) adjon meg egy parancssori eszköz és a Resource Manager-sablonok, amely leegyszerűsíti az üzembe helyezés Azure-erőforrások készletét.

## <a name="lack-of-visibility-into-system-issues"></a>Megismerheti a rendszerproblémák hiánya

**Technikai kockázati**: Nincs elegendő monitorozást és diagnosztikát az üzleti rendszerekben megakadályozhatja, hogy műveleteket személyek problémák azonosítása és javítása előtt a rendszer szolgáltatáskimaradás következik be, és jelentősen csökkenti a kimaradás megfelelően megoldásához szükséges idő.

**Házirendutasítás**: A következő házirendek hajtják végre:

- Fő mérőszámai és diagnosztikái mértékek azonosítva lesz az összes éles rendszerekre és összetevőit, és monitorozási és diagnosztikai eszközöket ezekben a rendszerekben a alkalmazni és műveleti személyzet által rendszeresen figyelni.
- Műveletek figyelembe veszi, monitorozási és diagnosztikai eszközök használatával nem éles környezetekben, például átmeneti és QA rendszerrel kapcsolatos problémák azonosítására, mielőtt bekövetkeznének az éles környezetben.

**A lehetséges tervezési lehetőségek**: [Az Azure Monitor](/azure/azure-monitor/), mely is tartalmazza, a Log Analytics és az Application Insights, az eszközöket biztosít gyűjtése és elemzése a telemetriai adatok segítségével megismerheti, hogyan az alkalmazások hajt végre, és proaktív módon azonosíthatja a problémák őket érintő és a az erőforrások függenek.

## <a name="configuration-security-reviews"></a>Konfiguráció biztonsági ellenőrzések

**Technikai kockázati**: Az idő múlásával új biztonsági fenyegetések vagy problémákat növelheti az erőforrások védelme a jogosulatlan hozzáférés kockázatát.

**Házirendutasítás**: Felhőalapú Cégirányítási folyamatok tartalmaznia kell a negyedéves tekintse át a konfigurációs felügyeleti csapataival rosszindulatú vagy az, hogy meg kell akadályozni a felhő-eszköz konfigurációja használati minták azonosításához.

**A lehetséges tervezési lehetőségek**: Negyedéves biztonsági felülvizsgálat értekezlet, amely tartalmazza a cégirányítási csapat tagjai és a konfiguráció a felhőalapú alkalmazások és erőforrások felelős informatikai munkatársak létesíteni. Tekintse át a meglévő biztonsági adatok és metrikák létrehozására, hézagok jelenlegi üzembe helyezési gyorsítás szabályzat azokat az eszközöket és a frissítési szabályzat minden olyan új kockázatok csökkentése érdekében.

## <a name="next-steps"></a>További lépések

A cikkben említett kiindulási pontként minták segítségével házirendeket, amelyek igazodnak a felhő bevezetésének csomagok adott üzleti kockázatok auditálására fejlesztése.

A saját egyéni házirend-utasítások identitáskezeléshez kapcsolódó fejlesztésének megkezdéséhez töltse le a [identitásra építve sablon](template.md).

-K gyorsabb elfogadása, ezen a területen, válassza a [gyakorlatban hasznosítható Cégirányítási utazás](../journeys/overview.md) , hogy a legtöbb szorosan igazodik a környezetében. Ezután módosítsa a kialakítás révén az adott vállalati házirendet érintő döntések.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md)