---
title: 'CAF: Üzembe helyezési gyorsítás metrikákat, a mutatókat és a szervezet kockázattűrési határát'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Üzembe helyezési gyorsítás metrikákat, a mutatókat és a szervezet kockázattűrési határát
author: alexbuckgit
ms.openlocfilehash: 87d6f9f67b98d5761aced560392c9d38fa682ee7
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899167"
---
# <a name="deployment-acceleration-metrics-indicators-and-risk-tolerance"></a>Üzembe helyezési gyorsítás metrikákat, a mutatókat és a szervezet kockázattűrési határát

Ebből a cikkből segítséget nyújt az üzleti szervezet kockázattűrési határát számszerűsítik a központi telepítési gyorsítás vonatkozik. Metrikák és a mutatók segítségével hozhat létre egy üzleti esetet, befektetés végzett az adatbázisplatform működését is az üzembe helyezés gyorsítás fegyelem.

## <a name="metrics"></a>Mérőszámok

Üzembe helyezés gyorsítás telepítése, frissítése és megfelelő rendszerek művelet konfigurált felhőbeli erőforrások karbantartása összpontosít. A következő információkat akkor hasznos, ha a felhő irányítási fegyelem bevezetése:

- **A helyreállítási időre vonatkozó célkitűzés (RTO)**. A maximális elfogadható idő, amely egy incidens után egy alkalmazás elérhetetlen maradhat.
- **Helyreállítási időkorlát (RPO)**. A maximális időtartamot, katasztrófa során elfogadható adatvesztés. Ha például egyetlen adatbázisban tárol adatokat, és nem készít replikát egy másik adatbázisba, és óránként végez biztonsági mentéseket, akkor legfeljebb egy órányi adatot veszíthet.
- **Átlagos idő (MTTR) helyreállításához**. Állítsa vissza egy összetevő egy meghibásodás után szükséges átlagos időt.
- **(MTBF) hibák közötti átlagos idő**. Az időtartam, amely egy összetevő ésszerűen várható között valamilyen okból kimaradás lép futtatásához. Ez a metrika segítségével kiszámíthatja, hogy milyen gyakran egy szolgáltatás nem lesz elérhető.
- **Szolgáltatói szerződések (SLA)**. Ez magában foglalhatja mind a Microsoft kötelezettségvállalások üzemidejével és elérhetőségével az Azure-szolgáltatások, valamint vállalt a vállalat által belső és külső ügyfeleinek.
- **Üzembe helyezési idő**. Mennyi ideig telepíti központilag a frissítéseket egy meglévő rendszer szükséges.
- **Eszközök out-az-megfelelőségi**. A szám vagy álló nem meghatározott szabályzatoknak való megfelelőségét.

## <a name="risk-tolerance-indicators"></a>Kockázati tolerancia mutatók

Üzembe helyezés gyorsítás kapcsolatos nagymértékben kapcsolódó száma és a vállalat számára üzembe helyezett felhőalapú rendszerek összetettségét. A felhő hagyatéki növekedésével a telepített rendszerek és a felhőbeli erőforrások frissítési gyakorisága növekszik. Erőforrások közti függőségeket magnify biztosítsa a megfelelő erőforrások konfigurálását és rendszerek rugalmasság tervezése, ha egy vagy több erőforrást váratlan állásidő fontosságát.

<!-- "en-us" location is required for the URL below. -->

Vegye figyelembe a DevOps bevezetése vagy [DevSecOps](https://www.microsoft.com/en-us/securityengineering/devsecops) szervezeti kultúra korai szakaszában a cloud bevezetési folyamata. A hagyományos vállalati informatikai szervezetek gyakran kell ügyfelek műveletek, a biztonság és a fejlesztői csapatok, amelyek gyakran nem jól működhet, vagy akár módosíthassák vagy megerősítve a rosszindulatú felé egy másik. Ezek a kihívások korai FELISMERVE, és a csapatok minden főbb érintettek integrálása biztosítja a felhőre való áttérés közben biztonságos és megfelelően szabályozott mozgékonyság.

A DevSecOps csapata és üzleti résztvevőkkel azonosításához együttműködve [üzleti kockázatai](business-risks.md) konfigurációval kapcsolatos, majd vizsgálja meg, az elfogadható alapkonfigurációjához tartozó konfigurációs szervezet kockázattűrési határát. Ebben a szakaszban CAF útmutatás példákat talál, de a részletes kockázatokat és a vállalat vagy a központi telepítések alapterveit valószínűleg eltérőek lehetnek.

Ha már van egy alapkonfiguráció elfogadhatatlan mértékű növekedést az azonosított kockázatok a minimális referenciaalapokhoz képest történő létrehozásához. Ezek referenciaalapokhoz képest történő eseményindítók a szerepét, amikor szüksége van arra kockázatok csökkentése érdekében. A következő néhány példa a konfigurációhoz kapcsolódó hogyan mérőszámokat, például a fent tárgyalt, a központi telepítési gyorsítás fegyelem megnövekedett befektetést igazolja.

- **Szolgáltatásiszint-szerződés (SLA) eseményindító**. Egy vállalat, amely a külső ügyfelek esetében, és a belső partnerei az SLA nem felel meg a központi telepítési gyorsítás fegyelem csökkentése érdekében a rendszer-állásidő kell befektetni.
- **Helyreállítási idő eseményindítók**. Ha egy vállalati szükséges küszöbértékek meghaladja a helyreállítási idő a következő rendszerhiba, azt kell fektet hozzájárulhat a központi telepítési gyorsítás szakágankénti és rendszerek kialakításakor csökkentheti vagy elkerülheti a hibákat, vagy az egyes összetevők állásidő hatását.
- **Konfigurációs eltéréseket eseményindítók**. A vállalat, amelyen jelentkezik a váratlan módosítások kulcs rendszerösszetevők konfigurációjában, és a központi telepítését vagy a frissítéseket, a rendszer a hibák kell beruházni a központi telepítési gyorsítás fegyelem kiváltó okokért és szervizelési lépések azonosításához.  
- **Megfelelőségi eseményindítók kívül**. Ha megfelelőségi kívüli erőforrások száma meghaladja a megadott küszöbértéket (akár erőforrások teljes száma, vagy az erőforrások teljes százalékaként), egy vállalati üzembe helyezés gyorsítás fegyelem fejlesztései az egyes erőforrások biztosításához kell beruházni konfigurációs a megfelelőség kialakításához az adott erőforrás életciklusának marad.
- **A projekt ütemezési eseményindítókról**. Ha a vállalati erőforrások és alkalmazások telepítése gyakran az idő meghaladja a küszöbérték megadása, egy vállalati be kell bevezetni, vagy javíthatja az automatikus üzembe helyezést, a konzisztencia és a tervezhetőség érdekében üzembe helyezési gyorsítás folyamatainak. Napokban mért, vagy akár üzembe helyezéshez szükséges idő általában hetek jelzik optimálisnál rosszabb gyorsítás központi telepítési stratégiát.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-mérőszámok és hibatűrő mutatók, amelyek a jelenlegi cloud bevezetési tervet.

Building a kockázatokat és a hibatűrést, szabályozó kezelésének és központi telepítési gyorsítás szabályzat betartása egy folyamatot hoz létre.

> [!div class="nextstepaction"]
> [Szabályzat betartása folyamatok létrehozása](compliance-processes.md)
