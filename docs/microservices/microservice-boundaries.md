---
title: "Mikroszolgáltatási határok azonosítása"
description: "Mikroszolgáltatási határok azonosítása"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 046749191bd565813218b3834cb4674c4c5100e2
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-identifying-microservice-boundaries"></a>Mikroszolgáltatások tervezése: mikroszolgáltatási határok azonosítása

Mi az a mikroszolgáltatási megfelelő a mérete? Gyakran hallott valami hatását, "túl nagy, és nem túl kicsi" &mdash; , és ez biztosan megfelelő, amíg nincs nagyon hasznos a gyakorlatban. De ha egy gondosan tervezett tartománymodell, sokkal könnyebben mikroszolgáltatások létrehozására vonatkozó ok.

![](./images/bounded-contexts.png)

## <a name="from-domain-model-to-microservices"></a>A tartomány modell mikroszolgáltatások

Az a [előző fejezetben](./domain-analysis.md), meghatározott a dron továbbítási alkalmazást hozhat létre a kötött környezetek készlete. Azt tekintett jobban meg ezeket a kötött környezetben, amelyet szállítási környezetben futhat, és az entitások, összesítések, készletének azonosított és, hogy a tartományi szolgáltatásoktól, amelyet a környezetben.

Most még készen áll a modell az alkalmazás tervét. Íme egy megközelítés, amely segítségével a modell mikroszolgáltatások származik.

1. A kötött környezetben kezdődik. A funkció egy mikroszolgáltatási általában nem kell fedik egynél több kötött környezetben. Definíció a kötött környezetben a határ az egyes tartományokhoz-modell jelöli meg. Ha talál meg, hogy egy mikroszolgáltatási együtt keveri-e másik tartományba modellek, ez a jele, hogy visszalép, és a tartomány elemzés pontosítsa szeretne.

2. Ezután nézze meg a tartomány modell összesíti. Aggregátumok gyakran esetén használható jól mikroszolgáltatások létrehozására. Egy jól megtervezett összesítést mutat egy tetszetős mikroszolgáltatási jellemzői számos, mint:

    - Az aggregátumok üzleti követelmények, nem pedig például az adatok elérése vagy az üzenetkezelési technikai problémákat származik.  
    - Összesítő magas működési Kohéziós kell rendelkeznie.
    - Egy összesítésnek az adatmegőrzési szegélyére.
    - Aggregátumok lazán kell lennie. 
    
3. Tartományi szolgáltatások esetén is használható jól mikroszolgáltatások létrehozására. Tartományi szolgáltatások több összesítések állapotmentes műveletek legyenek. Egy tipikus példája az olyan munkafolyamatot, amely magában foglalja a több mikroszolgáltatások létrehozására. Példa erre a dron továbbítási alkalmazást hozhat létre a lesz látható.

4. Mindemellett érdemes lehet megfontolnia nem funkcionális követelményeinek. Nézze meg a tényezőket, mint a csapat mérete, adattípusok, technológiák, méretezhetőségi követelményeinek, rendelkezésre állási követelmények és biztonsági követelményeinek. Ezek a tényezők vezethet, hogy további egy mikroszolgáltatási felbontása két vagy több kisebb szolgáltatás, vagy ellenkező tegye, és több mikroszolgáltatások egy egységgé kombinálják. 

Miután azonosította a mikroszolgáltatások az alkalmazásban, ellenőrizze a kialakítás az alábbi feltételek alapján:

- Minden szolgáltatás egyetlen felelősségi rendelkezik.
- Nincsenek chatty hívások nem szolgáltatások között. Ha a felosztás két szolgáltatások funkciókat hatására túlságosan chatty is, ezeket a funkciókat tartozó ugyanazt a szolgáltatást a hiba lehet.
- Egyes szolgáltatások nem elég nagy, hogy azt épül fel egy kis csoport egymástól függetlenül működik-e.
- Nincsenek, amely szükséges két vagy több zárolás lépésben telepítendő szolgáltatások közötti függőségek. Mindig lehetővé kell tenni bármely egyéb szolgáltatások üzembe helyezésével szolgáltatás központi telepítése.
- Szolgáltatások szorosan nem összekapcsolt, és egymástól függetlenül lehet fejleszteni.
- A szolgáltatás határok nem hoz létre az adatok konzisztenciájának vagy integritási kapcsolatos problémák. Néha fontos funkciók üzembe egy egyetlen mikroszolgáltatási adatkonzisztencia karbantartásához. Említett, vegye figyelembe, hogy tényleg szüksége van az erős konzisztencia. Stratégiák a végleges konzisztencia címzési egy elosztott rendszerben, és a szolgáltatások gyakran decomposing előnyeit járó a végleges konzisztencia kezelése kihívásaival.

Mindenekelőtt fontos gyakorlati, és ne feledje, hogy tartományi-központú kialakítás iteratív folyamat. Ha kétségei vannak, a kiindulási pont további coarse-grained mikroszolgáltatások létrehozására. Egy mikroszolgáltatási felosztása két kisebb szolgáltatás még újrabontása funkciók között több meglévő mikroszolgáltatások könnyebb.
  
## <a name="drone-delivery-defining-the-microservices"></a>Dron kézbesítési: A mikroszolgáltatások meghatározása

Visszahívása, hogy a fejlesztői csapat a négy összesítések kellett azonosított &mdash; kézbesítési, a csomag, a dron és a fiók &mdash; és két tartományi szolgáltatásokban, az ütemező és a felügyelő. 

Kézbesítési és csomag esetén mikroszolgáltatások nyilvánvaló használható. Az ütemező és a felügyelő koordinálja a tevékenységét más mikroszolgáltatások, érdemes a tartományi szolgáltatások, mikroszolgáltatások végrehajtásához.  

Dron és fiók is érdekes, mert más kötött környezetekhez tartoznak. Egy beállítást a dron hívni az ütemező és fiókot, amelyet környezetek közvetlenül. Egy másik lehetőség, ha dron és a fiók mikroszolgáltatások belül a szállítási, amelyet a környezetben. Ezek mikroszolgáltatások volna résidőkiosztással API-k vagy az adatok sémákat, amely a szállítási környezetben érdemesebb jelentkezik, mintha a kötött környezetek között.

A dron és a fiókot, amelyet részleteit környezetek Ez az útmutató túlmutató vannak, ezért létrehoztunk utánzatait szolgáltatások számukra a hivatkozás megvalósításában. De az alábbiakban néhány tényezőket kell figyelembe venni ebben a helyzetben:

- Mi az a hálózata terhet hívása közvetlenül be a másik kötött környezetet? 

- Az adatok séma a más kötött környezet megfelelő-e ebben a környezetben, vagy azt jobban kell rendelkeznie a kötött környezet igényeinek megfelelő séma? 

- A másik kötött környezetet egy olyan korábbi rendszer? Ha igen, létrehozhat egy szolgáltatás, amely úgy működik, mint egy [elleni sérülés réteg](../patterns/anti-corruption-layer.md) lefordítani az örökölt rendszer és a modern alkalmazások között. 

- Mi az a csoport struktúra? Az egyszerű, a másik kötött környezetet felelős csapat kommunikálni? Ha nem, egy szolgáltatás, amely a két környezet közötti közvetítő létrehozása segíthet csoportközi kommunikációs költségeinek csökkentése érdekében.

Amennyiben azt nem veszi figyelembe nem működési vonatkozó követelményei. Az alkalmazás átviteli követelmények gondolat, a fejlesztői csapat lezárását hozzon létre egy külön adatfeldolgozást mikroszolgáltatási választásával dolgozhat fel ügyfélkérelmek felelős. A mikroszolgáltatási megvalósítandó [Terheléskiegyenlítés](../patterns/queue-based-load-leveling.md) által a bejövő kérések üzembe egy puffer feldolgozásra. A Feladatütemező a kérelmek beolvassa a pufferből, és a munkafolyamat végrehajtásához. 

Nem funkcionális követelményeinek a csapata egy további szolgáltatás létrehozásához vezetett. Az összes szolgáltatást, amennyiben az ütemezés és a valós idejű csomagok rendelkezik folyamattal kapcsolatos törölték. De a rendszer is minden szállítási előzmények tárolása a hosszú távú tárolás, az adatok elemzésére. A csapat számít, így ez a kézbesítési szolgáltatás feladata. Azonban az adattárolási követelmények eltérőek meglehetősen korábbi elemzéshez és üzenetsoroktól műveletek (lásd: [adatok szempontok](./data-considerations.md)). Ezért úgy döntött, a csapat külön kézbesítési előzmények szolgáltatás, amely a kézbesítési szolgáltatás DeliveryTracking események figyelését, és az eseményeket írni az hosszú távú tárolás létrehozása.

Az alábbi ábrán a Tervező ezen a ponton:
 
![](./images/microservices.png)

## <a name="choosing-a-compute-option"></a>A számítási lehetőség kiválasztása

A kifejezés *számítási* a számítási erőforrásokért, az alkalmazást futtató üzemeltetési modell hivatkozik. Egy mikroszolgáltatások architektúra két megközelítés különösen népszerű:

- A szolgáltatás az orchestrator dedikált csomópontok (VM) szolgáltatás kezeli.
- Egy kiszolgáló nélküli architektúra függvények használatával szolgáltatásként (FaaS). 

Ezek nem csak a beállítások, amíg azok mindkét bevált módszerekkel mikroszolgáltatások létrehozására. Egy alkalmazás tartalmazhat mindkét megközelítés.

### <a name="service-orchestrators"></a>Szolgáltatás orchestrators

Az orchestrator telepítésére és kezelésére a szolgáltatások kapcsolódó feladatokat végzi. Ezek a feladatok közé tartozik a csomópontok, szolgáltatások, a nem megfelelő szolgáltatások újraindítása állapotának figyelése szolgáltatásokban helyezi, és töltse be a hálózati forgalom terheléselosztása között szolgáltatáspéldány, a szolgáltatásészlelés, egy szolgáltatás példányának számának méretezését, és alkalmazása frissíteni a konfigurációt. Népszerű orchestrators Kubernetes, a DC/OS, a Docker Swarm és a Service Fabric tartalmazza. 

- [Az Azure Tárolószolgáltatás](/azure/container-service/) (ACS) az Azure-szolgáltatások, amely lehetővé teszi egy éles használatra kész Kubernetes, DC/OS- vagy Docker Swarm-fürt központi telepítése.

- [(Az Azure Tárolószolgáltatás) AKS](/azure/aks/) egy felügyelt Kubernetes szolgáltatás. AKS rendelkezések Kubernetes és tesz elérhetővé a Kubernetes API-végpontok, de használja, és kezeli a Kubernetes vezérlő vezérlősík hajt végre az automatikus frissítések automatikus javítás, automatikus skálázás és egyéb kezelési feladatok. Az eltolásokat tekintheti AKS, hogy a "Kubernetes API-k szolgáltatásként." Írásának időpontjában AKS még mindig preview. Azonban várható, hogy AKS lesz-e az előnyben részesített módja Kubernetes futtassa az Azure-ban. 

- [A Service Fabric](/azure/service-fabric/) egy elosztott rendszerek platformja csomagolása, üzembe helyezése és kezelése mikroszolgáltatások létrehozására. Mikroszolgáltatások létrehozására is telepíthető a Service Fabric, tárolók, bináris végrehajtható fájlok, vagy mint [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction). A Reliable Services programozási modell használatával szolgáltatások használhatja közvetlenül a Service Fabric programozási API-kat a rendszer, a jelentés állapotának lekérdezése, konfigurációs és a kód módosítására értesítések fogadása és egyéb szolgáltatások felderítése. A Service Fabric legfontosabb különbséget a használatával állapot-nyilvántartó szolgáltatások erős összpontosít [megbízható gyűjtemények](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

### <a name="containers"></a>Tárolók

Egyes esetekben személyek szolgáltatással kapcsolatban tárolók és mikroszolgáltatások létrehozására az ugyanaz, mintha. Amíg, amely nem igaz értékű &mdash; nem szükséges tárolók mikroszolgáltatások létrehozására &mdash; tárolók rendelkezik néhány előnyről szempontjából különösen mikroszolgáltatások, például:

- **Hordozhatósága**. A tároló lemezkép csomag egy különálló futtató anélkül, hogy telepítse a szalagtárak vagy más függőségek. Amely megkönnyíti az központi telepítéséhez. Tárolók megkezdődött, és így akkor is léptetési mentése új példányok további terhelés kezelése érdekében vagy csomóponthibák helyreállítása gyorsan, leállt. 

- **Sűrűség**. Tárolók olyan egyszerűsített képest a futó virtuális gépek esetén, mert ugyanazt az operációs rendszer-erőforrások. Amely lehetővé teszi a csomag több tároló alakzatot egy egycsomópontos, amely akkor különösen akkor hasznos, ha az alkalmazás számos kis szolgáltatásból áll.

- **Erőforrás-elkülönítést**. Korlátozhatja a memória- és elérhető a tárolóhoz, így győződjön meg arról, hogy egy elszabadult folyamat nem lefoglalhat a gazdagép-erőforrások mennyiségét. Tekintse meg a [válaszfal mintát](../patterns/bulkhead.md) további információt.

### <a name="serverless-functions-as-a-service"></a>Kiszolgáló nélküli (funkciók szolgáltatásként)

Egy kiszolgáló nélküli architektúrával nem a virtuális gépek vagy a virtuális hálózati infrastruktúra kezeléséhez. Ehelyett telepít kódot és az üzemeltetési szolgáltatás kezeli, ezt a kódot és egy virtuális Gépet, és futtatnia kell. Ez a megközelítés általában kis részletes funkciók, amelyek koordinált eseményvezérelt használó alkalmazást. Várólista elhelyezte alatt álló üzenet például előfordulhat, hogy vált egy függvényt, amely beolvassa a várólistából, és feldolgozza az üzenetet.

[Az Azure Functions] [ functions] egy kiszolgáló nélküli számítási szolgáltatás, amely támogatja a különböző függvény eseményindítók, beleértve a HTTP-kérelmekre, Service Bus-üzenetsorok, és az Event Hubs eseményeket. Teljes listáját lásd: [Azure Functions eseményindítók és kötések fogalmak][functions-triggers]. Fontolja meg is [Azure esemény rács][event-grid], ez az útválasztási szolgáltatás kezelt esemény az Azure-ban.

### <a name="orchestrator-or-serverless"></a>Az orchestrator vagy kiszolgáló nélküli?

Az alábbiakban néhány tényezők választás az orchestrator megközelítés és a kiszolgáló nélküli megközelítés során figyelembe kell venni.

**Kezelhetőségi** egy kiszolgáló nélküli alkalmazást könnyen kezelhető, mivel a platform kezeli az összes a számítási erőforrásokat meg. Az orchestrator kivonatolja a kezelése és konfigurálása a fürt egyes funkcióit, amíg azt nem teljesen elrejtése az alapul szolgáló virtuális gépeket. Az orchestrator szüksége lesz kell vennie, például terheléselosztás, Processzor- és memóriahasználatról, és a hálózati terhelést.

**Rugalmasság és a Vezérlés**. Egy orchestrator lehetővé teszi az ellenőrzése alatt tartja a konfigurálása és kezelése, a szolgáltatások és a fürt nagy mennyiségű. Mi a fontosabb nagyobb fokú összetettségével. Egy kiszolgáló nélküli architektúrával akkor feladták vezérlő bizonyos fokú mivel ezek az adatok azért.

**Hordozhatósága**. Az itt felsorolt orchestrators (Kubernetes, a DC/OS, Docker Swarm és a Service Fabric) összes futtatható helyi vagy több nyilvános felhőkben. 

**Alkalmazások integrálása**. Ez hozható létre olyan összetett alkalmazás használatával egy kiszolgáló nélküli architektúra kihívást jelenthet. Az Azure-ban egy lehetőség [Azure Logic Apps](/azure/logic-apps/) Azure Functions készletét. Ez a megközelítés példáért lásd: [, amely az Azure Logic Apps függvény létrehozása](/azure/azure-functions/functions-twitter-email.)

**Költség**. Az egy orchestrator kell fizetnie a a fürt futó virtuális gépeket. Egy kiszolgáló nélküli alkalmazást, a kell fizetnie csak a tényleges számítási erőforrások használatának. Mindkét esetben kell számításba a további szolgáltatások, például a tárolás, az adatbázisok és az üzenetkezelési szolgáltatások költségét.

**Méretezhetőség**. Az Azure Functions automatikusan méretezi kielégítéséhez, a bejövő események száma alapján. Az orchestrator, az ki lehet terjeszteni a fürtben futó szolgáltatás példányainak számát. A fürt további virtuális gépek hozzáadásával is méretezheti.

Az útmutató elsősorban használ Kubernetes, de volt használjuk az Azure Functions egy szolgáltatás, nevezetesen a kézbesítési előzmények szolgáltatás. Az Azure Functions remekül beválik, ha az adott szolgáltatáshoz, volt, mert az egy eseményvezérelt munkaterhelés. Az Event Hubs eseményindító használatával függvényt, a szolgáltatás szükséges kód mennyisége minimális. A kézbesítési előzmények szolgáltatás is nem a fő munkafolyamat azon része, is fut a Kubernetes fürtön kívüli nem befolyásolják a végpont felhasználó által kezdeményezett műveletek. 

> [!div class="nextstepaction"]
> [Adathozzáféréssel kapcsolatos szempontok](./data-considerations.md)

<!-- links -->

[acs-engine]: https://github.com/Azure/acs-engine
[acs-faq]: /azure/container-service/dcos-swarm/container-service-faq
[event-grid]: /azure/event-grid/
[functions]: /azure/azure-functions/functions-overview
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
