---
title: Tervezési minták felhőkhöz
titleSuffix: Azure Architecture Center
description: Tervezési minták megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.
keywords: Azure
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f0cc64c555be092f9efb51e1f769c3668ce281af
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641008"
---
# <a name="cloud-design-patterns"></a>Tervezési minták felhőkhöz

Ezek a tervezési minták hasznosak a megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.

Mindegyik minta ismerteti az általa kezelt problémát, a minta alkalmazásának szempontjait és egy, a Microsoft Azure-on alapuló példát. A legtöbb minta tartalmaz kódmintákat vagy kódrészleteket is, amelyek a minta Azure-on való alkalmazását mutatják be. A legtöbb minta ugyanakkor minden elosztott rendszeren alkalmazható, függetlenül attól, hogy az az Azure-on vagy más felhőplatformon üzemel-e.

## <a name="challenges-in-cloud-development"></a>A felhőalapú fejlesztésben rejlő kihívások

<!-- markdownlint-disable MD033 -->
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">Rendelkezésre állás</a></h3>
        <p>A rendelkezésre állás az időarány, amíg a rendszer működik és üzemel, általában az üzemidő százalékaként megadva. Befolyásolhatják rendszerhibák, infrastruktúraproblémák, rosszindulatú támadások vagy a rendszer terhelése.  A felhőalapú alkalmazások általában biztosítanak a felhasználó számára egy szolgáltatói szerződést (SLA), így az alkalmazásokat a rendelkezésre állás maximalizálására kell tervezni.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">Adatkezelés</a></h3>
        <p>Az adatkezelés a felhőalapú alkalmazások kulcsfontosságú eleme, és befolyásolja a legtöbb minőségi attribútumot. Az adatok általában különböző helyeken, több kiszolgálón találhatók a teljesítmény, a skálázhatóság vagy a rendelkezésre állás miatt, ez pedig különféle kihívásokat jelenthet. Fenn kell tartani például az adatok konzisztenciáját, és az adatokat jellemzően több különböző hely között kell szinkronizálni.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">Tervezés és implementálás</a></h3>
        <p>A jó tervezés olyan tényezőket is figyelembe vesz, mint a konzisztencia és a koherencia az összetevők tervezése és üzembe helyezése során, a karbantarthatóság az adminisztráció és a fejlesztés egyszerűsítéséhez, illetve az újrahasznosíthatóság, hogy az összetevők és alrendszerek más alkalmazásokban és más forgatókönyvekben is hasznosíthatók legyenek. A tervezés és az implementálás fázisában hozott döntések óriási hatással vannak a felhőalapú alkalmazások és szolgáltatások minőségére és teljes tulajdonlási költségére.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">Üzenetkezelés</a></h3>
        <p>A felhőalapú alkalmazások elosztott jellege miatt szükség van egy üzenetkezelési rendszerre is, amely összeköti az összetevőket és szolgáltatásokat, a skálázhatóság maximalizálása érdekében általában laza kapcsolással. Széles körben elterjedt az aszinkron üzenetkezelés, amely számos előnye mellett kihívásokat is tartogat, ilyen például az üzenetek rendezése, az ártalmas üzenetek kezelése, az idempotencia stb.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">Kezelés és figyelés</a></h3>
        <p>A felhőalapú alkalmazások egy távoli adatközpontban futnak, ahol nem lehetséges teljesen uralni az infrastruktúrát, vagy bizonyos esetekben, az operációs rendszert. Emiatt a kezelés és a figyelés bonyolultabb, mint egy helyszíni üzemelő példánynál. Az alkalmazásoknak elérhetővé kell tenniük a futásidőre vonatkozó adatokat, hogy a rendszergazdák és a kezelők ezek segítségével kezeljék és figyeljék a rendszert, valamint támogathassák a változó üzleti feltételeket és a testreszabást anélkül, hogy le kellene állítani vagy újra üzembe kellene helyezni az alkalmazást.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">Teljesítmény és skálázhatóság</a></h3>
        <p>A teljesítmény egy rendszer válaszkészségét mutatja egy művelet adott időtartamon belül való végrehajtásának vonatkozásában, a skálázhatóság pedig a rendszer azon képessége, hogy tudja-e kezelni a terhelés növekedését a teljesítmény romlása nélkül, vagy tudja-e azonnal növelni a rendelkezésre álló erőforrásokat. A felhőalapú alkalmazások általában különböző munkaterheléseknek és aktivitási csúcspontoknak vannak kitéve. Ezeket előre megjósolni, főleg egy több-bérlős forgatókönyvben, szinte lehetetlen. Az alkalmazásoknak inkább a horizontális felskálázásra kell képesnek lenniük bizonyos korlátok között, hogy megfeleljenek a csúcspont idején az igényeknek, és elvégezzék a horizontálisan leskálázást is, ha csökken az igény. A skálázhatóság nem csak a számítási példányokat érinti, de más elemeket is, például az adattárolást, az üzenetkezelési infrastruktúrát stb.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">Rugalmasság</a></h3>
        <p>A rugalmasság a rendszer azon képessége, hogy könnyedén kezelje a hibákat, és gyorsan helyreálljon. A felhőalapú üzemeltetés jellegéből (ahol az alkalmazások gyakran több-bérlősek, megosztott platformszolgáltatásokat használnak, versenyeznek az erőforrásokért és a sávszélességért, az interneten keresztül kommunikálnak, és hagyományos hardvereken futnak) következik, hogy nagyobb a valószínűsége, hogy átmeneti és állandóbb hibák is megjelennek. A hibák észlelése és a gyors és hatékony helyreállás szükséges a rugalmasság fenntartásához.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">Biztonság</a></h3>
        <p>A biztonságosság egy rendszer azon képessége, hogy meg tudja-e akadályozni a tervezett felhasználáson kívüli, rosszindulatú vagy véletlen műveleteket, és meg tudja-e előzni az információk nyilvánosságra kerülését vagy elvesztését. A felhőalapú alkalmazások elérhetők az interneten a megbízható helyszíni területek határain kívül is, gyakran nyilvánosak, és nem megbízható felhasználókat is kiszolgálhatnak. Az alkalmazásokat úgy kell megtervezni és üzembe helyezni, hogy védve legyenek a rosszindulatú támadások ellen, csak a jogosult felhasználók férhessenek hozzájuk, és védjék a bizalmas adatokat.</p>
    </td>
</tr>
</table>
<!-- markdownlint-disable MD033 -->

## <a name="catalog-of-patterns"></a>Mintakatalógus

|                                Mintázat                                |                                                                                                         Összegzés                                                                                                         |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [Ambassador](./ambassador.md)                     |                                                            Olyan segítő szolgáltatásokat hozhat létre, amelyek egy otthoni használatra szánt szolgáltatás vagy alkalmazás nevében küldenek hálózati kéréseket.                                                            |
|          [Anti-Corruption Layer](./anti-corruption-layer.md)          |                                                                  Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között.                                                                  |
|         [Backends for Frontends](./backends-for-frontends.md)         |                                                            Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre.                                                             |
|                       [Bulkhead](./bulkhead.md)                       |                                                        Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.                                                        |
|                    [Cache-Aside](./cache-aside.md)                    |                                                                                   Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból                                                                                    |
|                [Circuit Breaker](./circuit-breaker.md)                |                                                     Ha távoli szolgáltatáshoz vagy erőforráshoz csatlakozik, kezelheti azokat a hibákat, amelyek javítása esetleg sok időt venne igénybe.                                                     |
| [Claim Check](./claim-check.md) | A nagy méretű üzeneteket jogcímellenőrzésre és hasznos adatra oszthatja fel, hogy elkerülje az üzenetbusz túlterhelését. |
|       [Compensating Transaction](./compensating-transaction.md)       |                                                         Visszavonhat egy sorozatnyi, együttesen végül konzisztens műveletet meghatározó lépés által végrehajtott munkát.                                                         |
|            [Competing Consumers](./competing-consumers.md)            |                                                            Lehetővé teheti több párhuzamos felhasználó számára, hogy feldolgozzák az ugyanazon az üzenetkezelési csatornán fogadott üzeneteket.                                                             |
| [Compute Resource Consolidation](./compute-resource-consolidation.md) |                                                                        Egyetlen számítási egységbe konszolidálhat több feladatot vagy műveletet                                                                        |
|                           [CQRS](./cqrs.md)                           |                                                           Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.                                                            |
|                 [Event Sourcing](./event-sourcing.md)                 |                                                      Használhat egy csak hozzáfűzéssel bővíthető tárat az egy tartomány adatain elvégzett műveleteket leíró események teljes sorozatának rögzítésére.                                                      |
|   [External Configuration Store](./external-configuration-store.md)   |                                                           A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.                                                           |
|             [Federated Identity](./federated-identity.md)             |                                                                                A hitelesítést delegálhatja egy külső identitásszolgáltatónak.                                                                                |
|                     [Gatekeeper](./gatekeeper.md)                     | Védheti az alkalmazásokat és szolgáltatásokat egy dedikált üzemeltető példány segítségével, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között, érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat közöttük. |
|            [Gateway Aggregation](./gateway-aggregation.md)            |                                                                     Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.                                                                      |
|             [Gateway Offloading](./gateway-offloading.md)             |                                                                         A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.                                                                         |
|                [Gateway Routing](./gateway-routing.md)                |                                                                              Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.                                                                               |
|     [Health Endpoint Monitoring](./health-endpoint-monitoring.md)     |                                              Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök.                                               |
|                    [Index Table](./index-table.md)                    |                                                                Indexeket hozhat létre a lekérdezések által gyakran hivatkozott adattárbeli mezőkről.                                                                 |
|                [Leader Election](./leader-election.md)                |   Koordinálhat egy elosztott alkalmazásban az együttműködő feladatpéldányokból álló gyűjtemény által végrehajtott műveleteket, ha vezetőnek választ meg egy példányt, amely vállalja a többi példány kezelésével járó felelősséget.    |
|              [Materialized View](./materialized-view.md)              |                                        Létrehozhat előre kitöltött nézeteket egy vagy több adattár adataiból, ha az adatok formázása nem ideális a szükséges lekérdezési műveletekhez.                                        |
|              [Pipes and Filters](./pipes-and-filters.md)              |                                                        Egy összetett feldolgozást végrehajtó feladatot lebonthat különálló, újrahasznosítható elemek sorává.                                                        |
|                 [Priority Queue](./priority-queue.md)                 |                                 Priorizálhatja a szolgáltatásoknak küldött kéréseket úgy, hogy a magasabb prioritású kéréseket a rendszer gyorsabban fogadja és dolgozza fel, mint az alacsonyabb prioritásúakat.                                  |
| [Közzétevő/előfizető](./publisher-subscriber.md) | Engedélyezheti egy alkalmazás számára, hogy több érdeklődő fogyasztó számára aszinkron módon, a küldők és a fogadók összekapcsolása nélkül jelentsen be eseményeket. |
|      [Queue-Based Load Leveling](./queue-based-load-leveling.md)      |                                               Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.                                               |
|                          [Retry](./retry.md)                          |               Engedélyezheti egy alkalmazás számára a szolgáltatásokhoz vagy hálózati erőforrásokhoz való csatlakozáskor jelentkező előre jelzett, átmeneti meghibásodások kezelését egy korábban meghiúsult művelet transzparens módon való ismételt megkísérlésével.                |
|     [Scheduler Agent Supervisor](./scheduler-agent-supervisor.md)     |                                                              Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon.                                                               |
|                       [Sharding](./sharding.md)                       |                                                                           Egy adattárat horizontális partíció- vagy szilánkkészletté oszthat fel.                                                                            |
|                        [Sidecar](./sidecar.md)                        |                                                    Egy alkalmazás összetevőit külön folyamatban vagy tárolóban helyezheti üzembe, így elkülönítést és beágyazást biztosíthat.                                                     |
|         [Static Content Hosting](./static-content-hosting.md)         |                                                          A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.                                                           |
|                      [Strangler](./strangler.md)                      |                                            Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél.                                            |
|                     [Szabályozás](./throttling.md)                     |                                                 Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.                                                 |
|                      [Valet Key](./valet-key.md)                      |                                                        Jogkivonatot vagy kulcsot használhat, amely korlátozott közvetlen hozzáférést biztosít az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.                                                        |
