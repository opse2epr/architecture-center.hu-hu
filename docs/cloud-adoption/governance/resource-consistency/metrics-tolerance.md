---
title: 'CAF: Erőforrás-mérőszámok konzisztencia mutatók és kockázattűrés'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás-mérőszámok konzisztencia mutatók és kockázattűrés
author: BrianBlanchard
ms.openlocfilehash: 3a2561a6d1d81a6395bb256e921a7a26898b45a0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899331"
---
# <a name="resource-consistency-metrics-indicators-and-risk-tolerance"></a>Erőforrás-mérőszámok konzisztencia mutatók és kockázattűrés

Ebből a cikkből segítséget nyújt az üzleti szervezet kockázattűrési határát számszerűsítik erőforrás konzisztencia vonatkozik. Metrikák és a mutatók segítségével, befektetés végzett erőforrás konzisztencia tantárgy stabilizálódó üzleti eset létrehozása.

## <a name="metrics"></a>Mérőszámok

Az erőforrás konzisztencia fegyelem összpontosít az üzembe helyezést felhőalapú üzemeltetési kezelésével kapcsolatos kapcsolatos veszélyforrások. A kockázati részeként elemzési információkat gyűjthet, érdemes kapcsolódó az informatikai műveletek meghatározni, hogy mekkora kockázatot, arcfelismerés, és mennyire fontos befektetés erőforrás konzisztencia cégirányítási, hogy a tervezett felhőalapú környezetekben.

Mivel minden munkahely különböző üzemeltetési esetekben, de a következő elemek képviselő kockázattűrése az erőforrás konzisztencia területen belül kiértékelésekor összegyűjtendő metrikák hasznos példákat:

- **Felhőbeli számítógépadatok**. Felhőbeli üzembe helyezett erőforrások teljes száma.
- **Erőforrások címkézetlen**. Szükséges számviteli, üzletmenetre gyakorolt hatás vagy szervezeti címkék nélkül erőforrások számát.
- **Eszközök használaton**. Memória, Processzor, vagy hálózati képességeit amelyeknél minden alatt által használt erőforrások száma.
- **Megfogyatkoznak erőforrás**. Ahol memória, Processzor, vagy hálózati képességeit elfogytak a terhelés erőforrások számát.
- **Erőforrás-kor**. Idő óta az erőforrás utolsó telepített vagy módosították.
- **Szolgáltatás rendelkezésre állása**. Tényleges üzemidő felhőben futtatott számítási feladatok képest a várt üzemidő százalékos értéke.
- **Kritikus állapotban lévő virtuális gépek**. Száma üzembe helyezett virtuális gépek, ha az egyik vagy a kritikus fontosságú problémák észlelésekor, amely tartandó annak érdekében, hogy a normál működés visszaállítása.
- **Riasztások súlyosság szerint**. Riasztások súlyosság szerint, egy üzembe helyezett objektum teljes száma.
- **Nem megfelelő állapotú alhálózati hivatkozások**. Hálózati problémák léptek fel az erőforrások számát.
- **Nem megfelelő állapotú szolgáltatásvégpont**. Problémák a külső hálózati végpontok száma.
- **A felhőbeli szolgáltató a Service Health-események**. Hibákat okozhatnak, esetleg a felhőszolgáltató által okozott teljesítmény incidensek számát.
- **Rendszerállapot biztonsági mentését**. Aktívan szinkronizált biztonsági másolatok számát.
- **Helyreállítás állapota**. Sikeresen végrehajtott recovery-műveletek száma.

## <a name="risk-tolerance-indicators"></a>Kockázati tolerancia mutatók

Felhőplatformjain funkciók, amelyek lehetővé teszik az üzembe helyezési alapkészletével csapatok hatékonyan kis méretű központi telepítések felügyeletéhez szükséges további alapos tervezés nélkül, vagy dolgoz fel kínálnak. Ennek eredményeképpen kisebb fejlesztési/tesztelési vagy kísérleti első számítási feladatokat, amelyek tartalmazzák a felhőalapú eszközeinek viszonylag kis mennyiségű alacsony kockázat szintjét képviseli, és valószínűleg nem kell annyit átszállítást megakadályozzák a formális erőforrás konzisztencia-szabályzat.

Azonban a felhő hagyatéki méretének növekedésével az eszközök kezelésének bonyolultságát válik jóval nehezebb. További eszközöket a felhő képes azonosítani erőforrások tulajdonjogát, és hasznos vezérlő erőforrás minimalizálja a kockázatot, kritikus fontosságú lesz. A felhő további üzleti szempontból alapvető fontosságú számítási feladatait van telepítve, a hasznos üzemidő válik a kritikus fontosságú, és a szolgáltatás megszűnésének lehetséges költség meghaladása tolerancia gyorsan lecsökken.

A felhő bevezetésének korai szakaszában, működik az informatikai műveletek csapata és üzleti az érintettekkel való azonosításához [üzleti kockázatai](business-risks.md) kapcsolatos erőforrás konzisztencia, majd vizsgálja meg, az elfogadható alapértékeit a szervezet kockázattűrési határát. Ez a szakasza a CAF példákat talál, de a részletes kockázatokat és a vállalat vagy a központi telepítések alapterveit eltérő lehet.

Ha már van egy alapkonfiguráció elfogadhatatlan mértékű növekedést az azonosított kockázatok a minimális referenciaalapokhoz képest történő létrehozásához. Ezek referenciaalapokhoz képest történő eseményindítók a szerepét, amikor szüksége van arra kockázatok csökkentése érdekében. A következő néhány példa a hogyan működési mérőszámokat, például a fent tárgyalt, az erőforrás konzisztencia fegyelem megnövekedett befektetést igazolja.

- **Címkézés és a trigger elnevezési**. Egy cég több, mint az erőforrások nem rendelkező X a szükséges információk címkézés, vagy nem engedelmeskedik-e a elnevezési szabványait vegye figyelembe ezeknek a szabványoknak finomíthatja és következetes alkalmazásához őket a felhőben üzembe helyezett erőforrás konzisztencia tantárgy beruházó eszközök.
- **Erőforrások szükségesnél több erőforrással ellátott eseményindító**. Ha a vállalat több, mint X % rendszeresen használja az elérhető memória, Processzor, vagy hálózati képességeit, befektetés nagyon kis mennyiségű erőforrás konzisztencia tantárgy az eszközök az alábbi erőforrások használatának optimalizálása érdekében ajánlott.
- **Underprovisioned erőforrások eseményindító**. Ha a vállalat több, mint X % eszközök rendszeresen kimerítsék a rendelkezésre álló memória, Processzor, vagy hálózati képességeit, befektetés erőforrás konzisztencia tantárgy a legtöbb ajánlott annak biztosítására, hogy ezeknek az eszközöknek meg kell akadályozni a szolgáltatási erőforrások rendelkezik a megszakítások számát.
- **Erőforrás-kor eseményindító**. A vállalat több mint X erőforrások, amelyek nem lettek frissítve a keresztül X hónapok előnyökkel jár, befektetés az erőforrás konzisztenciát célzó fegyelem annak biztosítására, aktív erőforrást javított, kifogástalan állapotú, miközben elavult vagy más módon nem használt eszközök kivonásáról.  
- **Szolgáltatás rendelkezésre állási eseményindító**. Egy vállalat, amely tapasztalt x %-os üzemidő az üzleti szempontból alapvető fontosságú szolgáltatásokat, a szolgáltatás megbízhatóságát az erőforrás konzisztencia fegyelem kell befektetni.
- **Virtuális gép állapota eseményindító**. Egy vállalat, amely több, mint X %-a virtuális gépek egy kritikus problémát tapasztal, azonosíthatja a problémákat, és javíthatja a virtuális gépek stabilitási erőforrás konzisztencia tantárgy kell beruházni rendelkezik.
- **Hálózati egészségügyi eseményindító**. Egy vállalat, amely rendelkezik több mint X % alhálózatokat vagy csatlakozási problémákat tapasztal, amikor végpontok kell beruházni erőforrás konzisztencia tantárgy azonosítása és a hálózati problémák megoldására.
- **Biztonsági mentési lefedettség eseményindító**. Egy vállalat az X helyen naprakész biztonsági mentések nélkül kritikus fontosságú eszközök % mint annak biztosítása érdekében a konzisztens biztonsági mentési stratégiájának erőforrás konzisztencia tantárgy megnövekedett befektetés előnyeit.
- **Biztonsági mentési állapotának eseményindító**. A vállalat több mint X % hiba helyreállítási műveletek be kell azonosítani a problémákat a biztonsági mentés és a fontos erőforrások biztosítása az erőforrás konzisztencia fegyelem tapasztaló védettek.

A pontos mérőszámokat és eseményindítók használatával tudja mérni a szervezet kockázattűrési határát, és a szintet, befektetés erőforrás konzisztencia tantárgy lesz a szervezet specifikus, de a fenti példák szolgáljon a hasznos hozzászólás belül a felhő Cégirányítási alapja Csapat.  

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-mérőszámok és hibatűrő mutatók, amelyek a jelenlegi cloud bevezetési tervet.

Building a kockázatokat és a hibatűrést, erőforrás konzisztencia-szabályzat betartása szabályozó és olyamat hoz létre.

> [!div class="nextstepaction"]
> [Szabályzat megfelelőségi folyamat létrehozása](compliance-processes.md)
