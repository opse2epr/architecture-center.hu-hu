---
title: Mikroszolgáltatások határainak azonosítása
description: Mikroszolgáltatások határainak azonosítása.
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 961dd98081978c312ca6e0e347c66bef70f01624
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484193"
---
# <a name="designing-microservices-identifying-microservice-boundaries"></a>Mikroszolgáltatások tervezése: Mikroszolgáltatások határainak azonosítása

Mi az a megfelelő méretű, a mikroszolgáltatások? Gyakran Hallott valamit a hatását, "túl nagy és túl kicsi" &mdash; , és bár ez természetesen megfelelő, nem a gyakorlatban nagyon hasznos. De ha elindítja a gondosan megtervezett tartományi modell, sokkal egyszerűbb mikroszolgáltatás-alapú vonatkozó döntések meghozatalát.

![Kapcsolódó kontextusokat ábrája](./images/bounded-contexts.png)

## <a name="from-domain-model-to-microservices"></a>A tartományi modellben mikroszolgáltatások

Az a [előző fejezetben](./domain-analysis.md), a Drone Delivery alkalmazás kapcsolódó kontextusokat készletét meghatározott. A kapcsolódó kontextusokat, ezek a szállítási korlátozott környezet egyik jobban megvizsgálta és azonosítja az entitásokban, az összesítések, és a tartományi szolgáltatásokat, amelyek korlátozott környezet.

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
  
## <a name="drone-delivery-defining-the-microservices"></a>Drone Delivery: A mikroszolgáltatások meghatározó

Idézze, hogy a fejlesztői csapat kellett azonosítja a négy összesítések &mdash; kézbesítési, a csomag, a Drone és a fiók &mdash; és a két tartomány szolgáltatás, az ütemező és a felügyelő.

Teljesítés és a Csomagmegosztás deduplikációra nyilvánvaló mikroszolgáltatás-alapú. Az ütemező és a felügyelő koordinálja a tevékenységét más mikroszolgáltatás-alapú, ezért érdemes megvalósítani a ezeket domain servicesben mikroszolgáltatásokat.

Drónos és fiókot is érdekes, mert egyéb kapcsolódó kontextusokat tartoznak. Az egyik lehetőség az ütemező a Drone meghívásához és fiókot, amelyet környezetek közvetlenül. Egy másik lehetőség, a Drone és a fiók mikroszolgáltatások belül a szállítási korlátozott környezet létrehozásához. Ilyen mikroszolgáltatásokból lenne résidőkiosztással között a kapcsolódó kontextusokat, API-k és adatsémák, amely a szállítási címhez tartozó környezeti érdemesebb teszi elérhetővé.

A Drone, valamint a fiók, amelyet környezetekben ez az útmutató nem terjed vannak, így azok utánzatként funkcionáló szolgáltatások referenciaimplementáció létrehoztunk. De Íme néhány tényező ebben a helyzetben érdemes figyelembe venni:

- Mi az a hálózati többletterhelést okoz, közvetlenül a többi körülhatárolt kontextus-be irányuló hívás?

- Ideális a sémát a többi körülhatárolt kontextus az ebben a környezetben, vagy jobban szeretné, hogy egy sémát, amely a körülhatárolt kontextus személyre szabott?

- Az a többi körülhatárolt kontextus egy korábbi rendszer? Ha tehát, előfordulhat, hogy létrehoz egy szolgáltatást, amely indextáblaként egy [sérülésgátló réteg](../patterns/anti-corruption-layer.md) az átalakításra a régi rendszer és a modern alkalmazás között.

- Mi az a csapat struktúra? Az egyszerű, a többi körülhatárolt kontextus felelős csapat kommunikálni? Ha nem, egy szolgáltatás, amely a két környezet közötti közvetítő létrehozása segíthet közötti kommunikáció költségeinek csökkentése érdekében.

Az eddigi azt még nem tekinthető minden olyan nem funkcionális követelmények. Az alkalmazás átviteli sebességet megkövetelő mértékegységeként, a fejlesztői csapat úgy döntött, hogy hozzon létre egy külön Adatbetöltési mikroszolgáltatás, amely az ügyfél kérelmek feldolgozására. A mikroszolgáltatások megvalósítandó [Terheléskiegyenlítés](../patterns/queue-based-load-leveling.md) feldolgozásra a pufferbe írja a bejövő kérelmek. Az ütemező beolvassa a kérelmeket a buffer szolgáltatásból származó és a munkafolyamat végrehajtása.

Nem funkcionális követelmények a csapata egy kiegészítő szolgáltatás létrehozásához vezetett. A szolgáltatásokhoz, amennyiben az ütemezés és a csomagok továbbítása valós időben a folyamat lett. De a rendszer is kell tárolnia minden szállítás előzményei a data-elemzéshez hosszú távú tároláshoz. A csapata tekintett feladata a Licenctovábbítási szolgáltatása, ami. De az adattárolási követelmények is igen különböző az előzményadatokat és átvitel közben az operations (lásd: [adatok szempontok](./data-considerations.md)). Ezért a csapat úgy döntött, hogy hozzon létre egy külön szállítás előzményei szolgáltatás, amely DeliveryTracking események kézbesítés a szolgáltatás figyelésére, és írja az események, hosszú távú tároláshoz.

Az alábbi ábrán látható ezen a ponton a terv:

![Tervezési diagramja](./images/microservices.png)

## <a name="choosing-a-compute-option"></a>Számítási lehetőség kiválasztása

A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A mikroszolgáltatási architektúrában a két módszer különösen népszerű:

- A szolgáltatás az orchestrator által felügyelt dedikált csomópontok (virtuális gépek) futó szolgáltatásokat.
- A functions egy kiszolgáló nélküli architektúra használatával szolgáltatásként (FaaS).

Bár ezek nem az egyetlen lehetőség, mind a mikroszolgáltatások megközelítéseket. Egy alkalmazás tartalmazhat mindkét módszerénél.

### <a name="service-orchestrators"></a>Szolgáltatás vezénylők

Az orchestrator szolgáltatások használatával történő üzembe helyezéséről kapcsolódó feladatokat végzi. Ezek a feladatok közé tartozik például a szolgáltatások csomópontok,-szolgáltatások újraindítása a nem megfelelő állapotú szolgáltatások állapotának figyelése, terheléselosztási hálózati forgalom elosztását a szolgáltatáspéldány, a szolgáltatásészlelés, a szolgáltatáspéldányok számának méretezése és alkalmazása konfiguráció frissítéseit. Népszerű szervezőeszközökkel helyezheti őket a Kubernetes, a Service Fabric, DC/OS és Docker Swarm közé tartozik.

Az Azure platformon vegye figyelembe a következő beállításokat:

- [Az Azure Kubernetes Service](/azure/aks/) (AKS) egy olyan felügyelt Kubernetes szolgáltatás. Az AKS rendelkezések Kubernetes és közzéteszi a Kubernetes API-végpontokat, de üzemelteti, és felügyeli a Kubernetes vezérlősík hajt végre automatikus frissítésekre és automatikus javításokat, automatikus skálázást és más felügyeleti feladatokat. Is felfoghatók AKS, hogy a "Kubernetes API-k szolgáltatás."

- [A Service Fabric](/azure/service-fabric/) csomagolása, üzembe helyezése és kezelése a mikroszolgáltatások egy elosztott rendszerplatform. Mikroszolgáltatások Service Fabric telepíthetők, tárolók, bináris végrehajtható fájlok, vagy mint [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). A Reliable Services programozási modell révén szolgáltatások közvetlenül a Service Fabric programozási API-kat a rendszer, a jelentés állapotának lekérdezése, konfigurálásról és a kód módosításait az értesítések fogadásához és egyéb szolgáltatások észlelését. A Service Fabric legfontosabb különbséget az állapotalapú szolgáltatások használatával erős koncentrálhat [a Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

- [Az Azure Container Service](/azure/container-service/) (ACS) egy Azure-szolgáltatás, amely lehetővé teszi egy üzemkész DC/OS, Docker Swarm vagy Kubernetes-fürt üzembe helyezése.

  > [!NOTE]
  > Kubernetes ACS által támogatott, bár javasoljuk az AKS az Azure-on futó Kubernetes. Az AKS továbbfejlesztett felügyeleti funkciók és a költséget biztosít.

### <a name="containers"></a>Tárolók

Más személyek beszélni tárolók és mikroszolgáltatások ugyanaz, mintha. Bár ez nem igaz &mdash; nem szükséges tárolók, mikroszolgáltatások &mdash; tárolók rendelkezik, amely különösen a mikroszolgáltatások, mint például előnyöket:

- **Hordozhatóság**. Tárolórendszerkép csomag egy különálló futtató anélkül, hogy kódtárakat vagy egyéb függőségek telepítése. Így azok egyszerűen üzembe helyezhetők. Tárolók indíthatók és úgy regisztrálhat új példányokat további terhelés kezeléséhez, illetve a csomóponthibák gyorsan, leállt.

- **Sűrűségű**. Tárolók olyan egyszerűsített képest a futó virtuális gép, mert ugyanazt az operációs rendszer-erőforrásokat. Amely lehetővé teszi, hogy több tároló telepítését egyetlen csomópont, amely akkor különösen hasznos, ha az alkalmazást több kis szolgáltatásra áll.

- **Erőforrás-elkülönítést**. Korlátozhatja a mennyi memóriát és CPU, amely elérhető egy tárolóba, amely segít annak biztosításában, hogy egy elszabadult folyamat nem lefoglalhat gazdagép-erőforrásokat. Tekintse meg a [válaszfal minta](../patterns/bulkhead.md) további információt.

### <a name="serverless-functions-as-a-service"></a>Kiszolgáló nélküli (Functions szolgáltatás)

Az egy [kiszolgáló nélküli](https://azure.microsoft.com/solutions/serverless/) architektúra, nem kell felügyelnie a virtuális gépek vagy a virtuális hálózati infrastruktúra. Ehelyett kell telepíteni, kód és az üzemeltetési szolgáltatás kezeli a kód üzembe egy virtuális Gépet az alakzatot, és futtassa a jelentést. Ez a megközelítés általában kis részletes funkciók, amelyek koordinált eseményalapú eseményindítókat használó alkalmazást. Ha például egy üzenetet egy üzenetsorba való elhelyezni kívánt is kiválthatják a függvény, amely az üzenetsorból olvas, és feldolgozza az üzenetet.

[Az Azure Functions] [ functions] egy kiszolgáló nélküli számítási szolgáltatás, amely támogatja a különböző függvény eseményindítók, beleértve a HTTP-kérelmekre, Service Bus-üzenetsorok és az Event Hubs-események. Teljes listáját lásd: [Azure Functions eseményindítók és kötések fogalmak][functions-triggers]. Emellett érdemes lehet [Azure Event Grid][event-grid], ez egy felügyelt esemény-útválasztó szolgáltatás az Azure-ban.

<!-- markdownlint-disable MD026 -->

### <a name="orchestrator-or-serverless"></a>Az orchestrator vagy a kiszolgáló nélküli?

<!-- markdownlint-enable MD026 -->

Az alábbiakban néhány szempontot figyelembe kell venni egy orchestrator-módszert és a egy kiszolgáló nélküli megközelítés közötti választáshoz.

**Kezelhetőségi** kiszolgáló nélküli alkalmazás a könnyen kezelhető, mert a platform kezeli az összes a számítási erőforrásokat az Ön számára. Az orchestrator felügyelete és a egy fürt konfigurálása bizonyos aspektusainak kivonatolja, amíg azt nem teljesen rejt el az alapul szolgáló virtuális gépeket. Az orchestrator-kell kapcsolatos problémák, például a terheléselosztás, a CPU és memória-felhasználás, és a hálózati terhelés.

**Rugalmasság és irányítás**. Az orchestrator felett konfigurálása és kezelése a szolgáltatások és a fürt nagy fokú biztosítja. Az egyensúlyt a további összetettséget. Kiszolgáló nélküli architektúrával adjon be néhány magas fokú felügyeletet mivel ezeket az adatokat emeli ki.

**Hordozhatóság**. Az itt felsorolt vezénylők (Kubernetes, DC/OS, Docker Swarm és a Service Fabric) is futtathatja a helyszíni vagy több nyilvános felhőkben.

**Alkalmazásintegráció**. Ez egy-egy kiszolgáló nélküli architektúra használatával összetett alkalmazások kihívást jelenthet. Az Azure-ban egy lehetőség [Azure Logic Apps](/azure/logic-apps/) az Azure Functions egy készletét. Erre a megközelítésre példa: [Azure Logic Apps szolgáltatással integrálható függvények létrehozása](/azure/azure-functions/functions-twitter-email).

**Költség**. Az orchestrator-fizet a virtuális gépek, amelyek a fürtön futtatják. Az egy kiszolgáló nélküli alkalmazást akkor kell fizetnie csak a tényleges számítási erőforrások felhasználásának. Mindkét esetben kell figyelembe vennie a költség, a további szolgáltatások, például a tárolás, adatbázisok és üzenetküldési szolgáltatások.

**Méretezhetőség**. Az Azure Functions automatikusan méretezi az igényeknek, a bejövő események száma alapján. Az orchestrator-horizontálisan növelje a fürtben futó szolgáltatás példányainak számát. A fürthöz további virtuális gépek hozzáadásával is méretezheti.

Referenciaimplementáció elsősorban használja a Kubernetes, de fejeződött használjuk az Azure Functions egy szolgáltatáshoz, azaz a szállítás előzményei szolgáltatás. Az Azure Functions remekül beválik, ha az adott szolgáltatáshoz, volt, mert egy eseményvezérelt munkaterhelés. A függvény meghívása egy Event Hubs-trigger használatával, a szolgáltatás szükséges minimális mennyiségű kódot. Emellett a szállítás előzményei szolgáltatás nem képezi részét a fő munkafolyamat, a Kubernetes-fürt-on kívül futnak, nem befolyásolja a felhasználó által kezdeményezett műveletek végpontok közötti késését.

> [!div class="nextstepaction"]
> [Az adatok kapcsolatos szempontok](./data-considerations.md)

<!-- links -->

[acs-engine]: https://github.com/Azure/acs-engine
[acs-faq]: /azure/container-service/dcos-swarm/container-service-faq
[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
