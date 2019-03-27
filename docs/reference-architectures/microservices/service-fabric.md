---
title: A Mikroszolgáltatási architektúra az Azure Service Fabricban
description: Üzembe helyezése az Azure Service fabric egy mikroszolgáltatás-alapú architektúra
author: PageWriter-MSFT
ms.date: 3/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 228e3242ce9f1ca45bc15c5b34f770bfc092dd43
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298989"
---
# <a name="microservices-architecture-on-azure-service-fabric"></a>A Mikroszolgáltatási architektúra az Azure Service Fabricban

Ez a referenciaarchitektúra üzembe helyezett Azure Service fabric a mikroszolgáltatási architektúra látható. A legtöbb telepítés esetén a kiindulási pont lehet alapszintű fürtbeállításokat mutatja.

![A Service Fabric a referencia-architektúra](./_images/ra-sf-arch.png)

> [!NOTE]
> Ez a cikk elsősorban a [Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction) Service Fabric programozási modell révén. A Service Fabric segítségével üzembe helyezheti és kezelheti [tárolók](/azure/service-fabric/service-fabric-containers-overview) van ez a cikk nem foglalkozik.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll. Egyéb feltételeket, lásd: [Service Fabric a terminológia áttekintése](/azure/service-fabric/service-fabric-technical-overview).

**Service Fabric-fürt**. A hálózaton keresztül csatlakozó készlete, virtuális gépek (VM), amelybe mikroszolgáltatásokat helyezhet üzembe és felügyelhet.

**A Virtual machine scale sets**. Virtuálisgép-méretezési csoportok létrehozása és kezelése egy csoportot az azonos, elosztott terhelésű és automatikus skálázású virtuális gépek lehetővé teszik. A tartalék és frissítési tartományokba is tartalmazza.

**Csomópontok**. A csomópontok használata a Service Fabric-fürthöz tartozó virtuális gépeket.

**A csomóponttípusok**. A csomópont típusa egy virtuális gép méretezési csoportot, amely üzembe helyezi a csomópontok gyűjteményei, jelöli. Service Fabric-fürt legalább egy csomópont típussal rendelkezik. Egy fürtben több csomóponttípus az, egy deklarálni kell a [elsődleges csomóponttípus](/azure/service-fabric/service-fabric-cluster-capacity#primary-node-type). Az elsődleges csomópont írja be a fürt fut a [Service Fabric-rendszerszolgáltatások](/azure/service-fabric/service-fabric-technical-overview#system-services). Ezek a szolgáltatások adja meg a Service Fabric platform képességeit. Az elsődleges csomóponttípushoz is funkcionál, a [magcsomópont](/azure/service-fabric/service-fabric-disaster-recovery#random-failures-leading-to-cluster-failures) a fürthöz, melyek a csomópontokat, melyek fenntartják a mögöttes fürt rendelkezésre állását. Konfigurálása [további csomóponttípusok](/azure/service-fabric/service-fabric-cluster-capacity#non-primary-node-type) a szolgáltatások futtatásához.

**Szolgáltatások**. A szolgáltatás hajtja végre a különálló függvény, amely elindíthatja és egyéb szolgáltatások függetlenül futnak. Példányok szolgáltatások a fürtben található csomópontok üzembe lesznek helyezve. Van a Service Fabric-szolgáltatás két fajta:

- **Az állapotmentes szolgáltatás**. Állapotmentes szolgáltatás nem tárol a szolgáltatás állapotát. Ha állapot megőrzését szükség, majd állapot írt és beolvasni egy külső áruházból, például az Azure Cosmos DB.
- **Állapotalapú szolgáltatás**. A [szolgáltatás állapota](/azure/service-fabric/service-fabric-concepts-state) fog zajlani a szolgáltatás. Legtöbb állapotalapú szolgáltatások révén a Service Fabric megvalósításának [a Reliable Collections](/azure/service-fabric/service-fabric-reliable-services-reliable-collections).

**Service Fabric Explorer**. [A Service Fabric Explorer] [ sfx] egy nyílt forráskódú eszköz vizsgálatához és kezeléséhez a Service Fabric-fürtöket.

**Az Azure folyamatok**. [A folyamatok](/azure/devops/pipelines/?view=azure-devops) része [Azure DevOps-szolgáltatásokkal](/azure/devops/index?view=azure-devops) és automatizált buildekig, tesztek és központi telepítések futtatja. Külső CI/CD megoldásokra, például a Jenkins is használhatja.

**Az Azure Monitor**. [Az Azure Monitor](/azure/azure-monitor/) összegyűjti és tárolja a metrikák és naplók, beleértve a platform az Azure-szolgáltatások mérőszámait a megoldás és alkalmazások telemetrikus adatait. Ezen adatok segítségével figyelheti az alkalmazást, állítsa be a riasztások és az irányítópultok és hajtsa végre a kiváltó okok elemzése a hibák. Az Azure Monitor integrálható a tartományvezérlők, csomópontok, és a tárolók, valamint a tároló naplóinak és fő csomópont naplóinak begyűjtése, hogy a Service Fabric.

**Azure Key Vault**. Használat [Key Vault](/azure/key-vault/) a mikroszolgáltatások, például kapcsolati karakterláncokat által használt alkalmazás titkos kulcsok tárolásához.

**Az Azure API Management**. Ebben az architektúrában [az API Management](/azure/api-management/api-management-key-concepts) API-átjáró, amely fogad az ügyfelektől érkező kéréseket, és továbbítja a szolgáltatások funkcionál.

## <a name="design-considerations"></a>Kialakítási szempontok

Ez a referenciaarchitektúra összpontosít [mikroszolgáltatás-architektúrákat](../../guide/architecture-styles/microservices.md). A mikroszolgáltatások általában a kód kisebb, független verziójú egysége. Szolgáltatás felderítési mechanizmusokon keresztül felderíthető és API-k keresztül kommunikálhat más szolgáltatásokkal. Mindegyik szolgáltatás önálló, és egyetlen üzleti képességet valósít meg. Hogyan bontható fel a doména aplikace mikroszolgáltatásokra kapcsolatos további információkért lásd: [tartományelemzés a mikroszolgáltatás-alapú modell használatával](../../microservices/model/domain-analysis.md).

A Service Fabric elkészítheti, telepítheti és mikroszolgáltatások hatékonyan frissítése infrastruktúrát biztosít. Automatikus méretezés, állapot kezelése, állapotának monitorozása és hiba esetén szolgáltatások újraindítása lehetőségeket is kínál.

A Service Fabric-alkalmazásmodell követi, ahol egy alkalmazás mikroszolgáltatások gyűjteménye. Az alkalmazás leírását egy [alkalmazásjegyzék](/azure/service-fabric/service-fabric-application-and-service-manifests) fájlt, amely meghatározza a különböző típusú szolgáltatás állapotadatait az adott alkalmazáshoz, és független szolgáltatási csomagokra mutató hivatkozások. Az alkalmazáscsomag általában is szolgálhat a felülbírálja az egyes beállítások a szolgáltatások által használt paramétereket tartalmaz. Minden szolgáltatás csomagnak jegyzékfájl, amely ismerteti a fizikai fájlok és mappák, a szolgáltatáshoz, beleértve a bináris fájlokat, a konfigurációs fájlok és a csak olvasható adatok, hogy a szolgáltatás futtatásához szükséges. Szolgáltatások és alkalmazások egymástól függetlenül rendszerverzióval ellátott, bővíthető.

Szükség esetén az alkalmazás jegyzékfájlja szolgáltatásokat, automatikusan hozzáférést kapnak az alkalmazás egy példányát létrehozásakor is ismertetik. Ezek az úgynevezett alapértelmezett szolgáltatások. Ebben az esetben az alkalmazásjegyzék is ismerteti, hogyan ezeket a szolgáltatásokat kell létrehozni, beleértve a szolgáltatás nevét, példányszámot, biztonsági vagy elkülönítési szabályzat és elhelyezési korlátozások.

> [!NOTE]
> Ne használja az alapértelmezett szolgáltatások, ha szeretné szabályozni a szolgáltatások teljes élettartama. Alapértelmezett szolgáltatások jönnek létre az alkalmazást a létrehozása, és futtassa az alkalmazást futtató is.

További információ a Service Fabric ismertetése: [ezért ismerje meg a Service Fabric?](/azure/service-fabric/service-fabric-content-roadmap)

### <a name="choose-an-application-to-service-packaging-model"></a>Az alkalmazás-to-service csomagolási modell kiválasztása

Mikroszolgáltatások alaptétele, hogy egyes szolgáltatások egymástól függetlenül is telepíthető. A Service Fabricben Ha csoport összes szolgáltatását egyetlen alkalmazáscsomag be, és a egy szolgáltatás meghibásodik, a frissítéshez a teljes alkalmazás frissítés lekérdezi vonva, amely megakadályozza, hogy a másik szolgáltatás frissítését.

A mikroszolgáltatási architektúrákban, ezért azt javasoljuk, több alkalmazáscsomagot. Egy vagy több szorosan kapcsolódó szolgáltatási típus helyezzen egy egyetlen alkalmazás típusát. Ha a csapat felelős a szolgáltatások, amelyek ugyanannyi ideig futnak, és a egy időben, frissíteni kell az azonos életciklussal rendelkezik, vagy erőforrások, például a függőségeket vagy a konfiguráció megosztása, majd jelölje ezen szolgáltatások típusok ugyanabból.

### <a name="service-fabric-programming-models"></a>A Service Fabric programozási modelljei

Mikroszolgáltatások Service Fabric-alkalmazások hozzáadásakor döntse el, hogy állapot vagy a magas rendelkezésre állású és megbízható tenni azok rendelkezik. Ha igen, azt adatot tárolhat kívülről vagy az adatokat a szolgáltatás részeként tartalmazza? Válassza ki a állapotmentes szolgáltatás, ha nem szeretne adatokat tárolni, vagy szeretné tárolni az adatokat a külső tárhelyen található. Ha meg szeretné-e tartani az állapot vagy az adatok a szolgáltatás részeként (például kell, hogy az adatok a memória, a kód közelében található), vagy nem egy függőség egy külső tároló a működését, fontolja meg az állapotalapú szolgáltatás kiválasztása.

Ha rendelkezik meglévő kódot, amely a Service Fabric szolgáltatásban futtatja, futtathatja azt egy Vendég végrehajtható, mint egy tetszőleges végrehajtható fájl, amely szolgáltatásként fut, vagyis. Másik lehetőségként csomagot készíthet a futtatható egy tároló, amely rendelkezik a telepítéshez szükséges összes függőséget. A Service Fabric állapotmentes szolgáltatások, tárolókkal és vendégalkalmazásokkal egyaránt modellek. A modell kiválasztásával kapcsolatban, tekintse át [Service Fabric programozási modell áttekintése](/azure/service-fabric/service-fabric-choose-framework).

Futtatható vendégalkalmazás, az Ön felelős a környezet, amelyben fut. karbantartása. Tegyük fel, hogy egy Vendég végrehajtható fájl futtatásához szükséges Python. Ha a végrehajtható fájl nem önálló, győződjön meg arról, hogy a környezet előzetesen már telepítve-e a szükséges Python-verzió van szükség. A Service Fabric nem felügyeli a környezetben. Az Azure kínál a környezetet, beleértve az egyéni virtuálisgép-rendszerképek és -bővítmények többféle módszerét.

További információkért lásd:

- [Alkalmazás becsomagolása](/azure/service-fabric/service-fabric-package-apps)
- [Csomagolása és üzembe helyezése egy meglévő végrehajtható fájlt a Service Fabric](/azure/service-fabric/service-fabric-deploy-existing-app)

### <a name="api-gateway"></a>API-átjáró

Egy [API-átjáró](../..//microservices/design/gateway.md) (bejövő) külső ügyfelek és a mikroszolgáltatások között helyezkedik el. Fordított proxy, mikroszolgáltatás-alapú ügyfelek útválasztási kérelmeit funkcionál. Emellett végrehajthatnak, például a hitelesítést, az SSL-lezárást és a sebességkorlátozás különböző általános feladatok.

Az Azure API Management a legtöbb esetben ajánlott, de [Træfik](https://docs.traefik.io/) népszerű nyílt forráskódú jelent. Mindkét technológia lehetőségek integrálva vannak a Service Fabric.

- Az API Management tesz közzé egy nyilvános IP-cím és útvonalak forgalmat az Ön felhőszolgáltatásainak. A Service Fabric-fürt ugyanazon a virtuális hálózaton a kijelölt alhálózatot fusson.  férhetnek hozzá a szolgáltatásokhoz a csomópont típusa, amely ki van téve a magánhálózati IP-címmel a terheléselosztón keresztül. Ez a beállítás csak az API Management fejlesztői és a prémium szintű csomagban érhető el. Az éles számítási feladatokhoz a prémium szint használja. Díjszabási információk leírt [az API Management díjszabása](https://azure.microsoft.com/pricing/details/api-management/). További információkért tekintse meg a Service Fabric és [Azure API Management áttekintése](/azure/service-fabric/service-fabric-api-management-overview).
- Træfik útválasztást, nyomon követését, naplók és mérőszámok funkciókat támogatja. Træfik állapotmentes szolgáltatás a Service Fabric-fürtben fut. Service versioning az Útválasztás nem használható. A szolgáltatás bejövő forgalom, mint a fürtben a fordított proxy Træfik beállítása további információkért lásd: [Azure Service Fabric-szolgáltató](https://docs.traefik.io/configuration/backends/servicefabric/). A Service Fabric Træfik használatával kapcsolatos további információkért lásd: [intelligens útválasztás a Service Fabric és Træfik](https://blogs.msdn.microsoft.com/azureservicefabric/2018/04/05/intelligent-routing-on-service-fabric-with-traefik/) (blogbejegyzés).

Más API-t Managementment lehetőségek a következők [Azure Application Gateway](/azure/application-gateway/) és [Azure bejárati ajtajának](/azure/frontdoor/). Ezek a szolgáltatások a feladatok végrehajtását, mint az útválasztást, az SSL-lezárást és a tűzfal az API Management szolgáltatással együtt is használható.

### <a name="interservice-communication"></a>Szolgáltatások közötti kommunikáció

Szolgáltatások közötti kommunikáció támogatására, fontolja meg a HTTP-n keresztül a kommunikációs protokollként. A legtöbb forgatókönyvhöz kiindulópontként, javasoljuk a [a fordított proxy szolgáltatás](/azure/service-fabric/service-fabric-reverseproxy) szolgáltatásfelderítési.

- Kommunikációs protokollt. Mikroszolgáltatási architektúrában a szolgáltatások kell egymással kommunikálni minimális kapcsoló futásidőben. Nyelvfüggetlen kommunikációhoz HTTP egy iparági szabványnak megfelelő az eszköz-és HTTP-kiszolgáló összes Service Fabric által támogatott különböző nyelveken elérhető számos. Ezért a HTTP helyett a Service Fabric beépített szolgáltatás távelérésének lehetővé tétele ajánlott a legtöbb számítási feladatokhoz.
- Szolgáltatás felderítése. Fürtön belüli más szolgáltatásokkal kommunikálni, egy ügyfél szolgáltatáshoz szükség van, oldja fel a célként megadott szolgáltatás jelenlegi helyet. A Service Fabric services helyezheti át csomópontokra, így dinamikusan változik, hogy a Szolgáltatásvégpontok között. Kapcsolatok végpontok elavult elkerülése érdekében a Service Fabric elnevezési szolgáltatásban használható lekérni frissített végpont-információkat. Azonban a Service Fabric is biztosít beépített [fordított proxy szolgáltatás](/azure/service-fabric/service-fabric-reverseproxy) , amely kivonatolja az elnevezési szolgáltatás. Ez a beállítás használata egyszerűbb és egyszerűbb kódot.

A szolgáltatások közötti kommunikációt eredményezhet más lehetőségek a következők,

- [Træfik](https://docs.traefik.io/) speciális útválasztást.
- [DNS](/azure/dns/) kompatibilitási forgatókönyvekhez, ahol a szolgáltatás DNS használatához vár.
- [ServicePartitionClient&lt;TCommunicationClient&gt; ](/dotnet/api/microsoft.servicefabric.services.communication.client.servicepartitionclient-1?view=azure-dotnet) osztály. Az osztály a Szolgáltatásvégpontok gyorsítótárazza, és engedélyezheti a jobb teljesítmény érdekében, közvetlenül a közvetítők és egyéni protokollokat nélkül szolgáltatások közötti hívások mennek.

## <a name="scalability-considerations"></a>Méretezési szempontok

Ezek az entitások fürt méretezése a Service Fabric támogatja:

- Méretezés az egyes csomóponttípusok csomópontok számát.
- Szolgáltatások méretezése.

Ez a szakasz az automatikus skálázás összpontosít. Ha szeretné, manuális méretezése helyzetekben, ahol szükséges. Ha például egy ahol manuális beavatkozásra van szükség, a példányok száma helyzetben.  

### <a name="initial-cluster-configuration-for-scalability"></a>A méretezhetőség érdekében a kezdeti fürtkonfigurációtól

Service Fabric-fürt létrehozásakor építhet ki a csomóponttípusok a biztonság és méretezhetőségi igényei alapján. Mindegyik csomóponttípus egymástól függetlenül skálázhatók, és a le van képezve egy virtuálisgép-méretezési csoportot.

- Hozzon létre egy csomópont típusa, amelyekhez különböző skálázhatósági vagy erőforrás-igényű szolgáltatásokat mindegyik csoportjához. Első lépésként kiépítése a csomópont típusa (válik, amely a [elsődleges csomóponttípus](/azure/service-fabric/service-fabric-cluster-capacity#primary-node-type)) a Service Fabric-szolgáltatások. Ezután hozzon létre külön csomóponttípusok a nyilvános vagy előtér-szolgáltatások futtatásához, és szükség esetén a háttérrendszer és a magán- vagy elkülönített szolgáltatások más csomóponttípusok. Adja meg [elhelyezési korlátozások](/azure/service-fabric/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies) úgy, hogy a szolgáltatások csak az importálni kívánt csomóponttípusok vannak telepítve.
- Adja meg az egyes csomóponttípusok tartóssági szint. A tartóssági szint arra, hogy a Service Fabric befolyásolhatja a virtual machine scale set-frissítések és karbantartási műveleteket az jelöli. Az éles számítási feladatokhoz válassza ki a Silver és magasabb szintű tartóssági szint. Az egyes csomagok kapcsolatos információkért lásd: [a fürt tartóssági jellemzőit](/azure/service-fabric/service-fabric-cluster-capacity#the-durability-characteristics-of-the-cluster).
- A bronz tartóssági szint használata esetén a bizonyos műveletekhez szükséges manuális lépéseket.  A csomóponttípusok a bronz tartóssági szint további lépésekre szükség a méretezés során. Műveletek méretezéssel kapcsolatos további információkért lásd: [Ez az útmutató](/azure/service-fabric/service-fabric-cluster-scale-up-down).

### <a name="scaling-nodes"></a>Csomópontok skálázása

A Service Fabric automatikus skálázás támogatja a horizontális leskálázási és a horizontális felskálázás. Mindegyik csomóponttípus egymástól függetlenül konfigurálhatók az automatikus skálázáshoz.

Mindegyik csomóponttípus legfeljebb 100 csomópont. Kisebb meg csomópontok kezdődhet, és adjon hozzá több csomópontot a betöltés függően. Ha több mint 100 csomópont a csomópont típusa van szüksége, szüksége lesz a további csomóponttípusok hozzáadása. További információkért lásd: [tervezési megfontolások a Service Fabric-fürt kapacitása](/azure/service-fabric/service-fabric-cluster-capacity). Egy virtuálisgép-méretezési csoportot nem azonnal méretezhető, ezért érdemes faktor, az automatikus skálázási szabályok beállítása során.

Támogatja az automatikus leskálázási, konfigurálnia kell a Silver vagy Gold tartóssági szint typ uzlu. Ez teszi arról, hogy a skálázás késik a Service Fabric-ig befejezett átvitel, szolgáltatások és, hogy a virtuálisgép-méretezési csoportok tájékoztatja a Service Fabric, hogy a virtuális gépek lesznek eltávolítva, nem csak le ideiglenesen.

A csomópont/fürt szintjén méretezésével kapcsolatos további információkért lásd: [méretezése az Azure Service Fabric-fürtök](/azure/service-fabric/service-fabric-cluster-scaling).

### <a name="scaling-services"></a>Szolgáltatások méretezése

Állapot nélküli és állapotalapú szolgáltatások többféle alkalmazni skálázás.

#### <a name="autoscaling-for-stateless-services"></a>Az automatikus skálázás az állapotmentes szolgáltatások esetében

- Az átlagos partíció terhelés eseményindítót használja. Ez az eseményindító határozza meg, hogy mikor van ellátva a szolgáltatás vagy a méretezési házirendben megadott betöltése küszöbérték alapján. Azt is beállíthatja, hogy milyen gyakran az eseményindító be van jelölve. Lásd: [partíció átlagos terhelés eseményindító példány-alapú méretezés](/azure/service-fabric/service-fabric-cluster-resource-manager-autoscaling#average-partition-load-trigger-with-instance-based-scaling). Ez lehetővé teszi, hogy az elérhető csomópontok számát a vertikális felskálázás.
- Állítsa be **InstanceCount** -1 értéket a szolgáltatásjegyzékben, amely tájékoztatja a Service Fabric a szolgáltatás egy példányának futtatását minden csomóponton. Ez a megközelítés lehetővé teszi, hogy a szolgáltatást, hogy a fürt skálázását követve rugalmasan méretezhető dinamikusan méretezhető. A fürt módosításait a csomópontok számát, mint a Service Fabric automatikusan hoz létre, és törli a szolgáltatáspéldányok megfelelően.

> [!NOTE]
> Bizonyos esetekben előfordulhat, hogy szeretné manuálisan méretezni a szolgáltatást.  Például ha egy szolgáltatás, amely beolvassa az Event Hubsból, érdemes egy dedikált példány minden egyes esemény hub-partícióról olvas, a partíció való egyidejű hozzáférés elkerülése érdekében.  

#### <a name="scaling-for-stateful-services"></a>Az állapotalapú szolgáltatások méretezése

Egy állapotalapú szolgáltatás méretezése vezérli a partíciók száma, mérete mindegyik partíció, és a egy adott gépen futó partíciók/replika száma.

- A particionált szolgáltatások létrehozásakor, célszerű megadnia, hogy nagy számú kis méretű partíciókat. Ha további csomópontokat ad hozzá, a Service Fabric alapértelmezés szerint osztja el a munkaterhelés összevonásához az új gépeket. Például ha 5 csomópontot és 10 partíciók, alapértelmezés szerint a Service Fabric fogja elhelyezni két elsődleges replika minden egyes csomóponton. Ha a csomópontok horizontális felskálázása, nagyobb teljesítményt érhet el, mert a munkahelyi egyenlően elosztva a további erőforrások. Ez a stratégia előnyeit forgatókönyvekkel kapcsolatos további információkért lásd: [méretezése Service Fabric](/azure/service-fabric/service-fabric-concepts-scalability).
- Partíciók hozzáadásával vagy eltávolításával nem is támogatott. A gyakran használt egy másik lehetőség méretezés dinamikusan létrehozása vagy törlése a szolgáltatások vagy a teljes alkalmazáspéldányok. Egy példa a minta leírt [méretezése a létrehozásával, vagy új elnevezett szolgáltatások eltávolítása](/azure/service-fabric/service-fabric-concepts-scalability#scaling-by-creating-or-removing-new-named-services).

További információkért lásd:

- [Méretezheti a Service Fabric-fürt a- vagy leskálázása automatikus skálázási szabályok használatával vagy manuálisan](/azure/service-fabric/service-fabric-cluster-scale-up-down)
- [A Service Fabric-fürt programozott skálázása](/azure/service-fabric/service-fabric-cluster-programmatic-scaling)
- [Egy virtuálisgép-méretezési csoportban hozzáadásával ki a Service Fabric-fürt méretezése](/azure/service-fabric/virtual-machine-scale-set-scale-node-type-scale-out)

### <a name="using-metrics-to-balance-load"></a>Mérőszámok segítségével a terhelés kiegyenlítése érdekében

Attól függően, hogyan tervezi meg a partíció lehetséges, hogy a többinél nagyobb forgalmat replikákkal rendelkező csomópontok. Ezen helyzet elkerülése érdekében particionálása a szolgáltatás állapotát, hogy az összes partíció között oszlanak meg azt. A particionálási séma egy megfelelő kivonatoló algoritmust használnak. Lásd: [– első lépések a particionálás](/azure/service-fabric/service-fabric-concepts-partitioning#get-started-with-partitioning).

A Service Fabric metrikák használatával tudja, hogyan helyezze el, és a egy fürtön belüli szolgáltatások elosztása. Megadhatja, hogy a szolgáltatás létrehozásakor szolgáltatáshoz társított minden egyes metrika alapértelmezett terhelést. A Service Fabric majd figyelembe veszi a terhelés, ha a szolgáltatás elhelyezése, illetve minden alkalommal, amikor a szolgáltatást (például frissítések) során áthelyezése a fürt csomópontjainak megpróbálni kell.

A szolgáltatás eredetileg megadott alapértelmezett terhelés élettartama során a szolgáltatás nem változik. Egy adott szolgáltatáshoz változó metrikákat rögzítheti, azt javasoljuk, hogy a szolgáltatás figyelése, és jelentést készít a terhelést dinamikusan. Ez lehetővé teszi, hogy módosítsa a foglalási egy adott időben a jelentett terhelés alapján a Service Fabric. Használja a [IServicePartition.ReportLoad](/dotnet/api/system.fabric.iservicepartition.reportload?view=azure-dotnet) metódus egyéni metrikákat jelenti. További információkért lásd: [dinamikus terhelésjelentés](/azure/service-fabric/service-fabric-cluster-resource-manager-metrics#dynamic-load).

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Helyezze el a szolgáltatásokat egy csomópont típusa nem az elsődleges csomóponttípushoz. A Service Fabric-rendszerszolgáltatások mindig az elsődleges csomóponttípushoz vannak telepítve. A szolgáltatások az elsődleges csomóponttípushoz van telepítve, ha ezeket előfordulhat, hogy az erőforrások rendszerszolgáltatások versenyeznek, és zavarja meg a helyrendszeri szolgáltatások. Typ uzlu várhatóan állapotalapú szolgáltatások működtetéséhez, hogy legalább öt csomópont-példányokat, és kiválasztja a Silver vagy Gold tartóssági szintet.

Vegye figyelembe, hogy beazonosítsa az erőforrások szolgáltatását. Lásd: [erőforrás cégirányítási mechanizmus](/azure/service-fabric/service-fabric-resource-governance#resource-governance-mechanism).

- Ne keverje a szabályozott erőforrás és erőforrás-nem tartozik az adott csomópont típusú szolgáltatások. A nem szabályozott szolgáltatások túl sok erőforrást, amely hatással van az erőforrás, szolgáltatás-szolgáltatások igénybe vehet. Adja meg [elhelyezési korlátozások](/azure/service-fabric/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies) , győződjön meg arról, hogy ilyen típusú szolgáltatások nem futnak a csomópontok ugyanazon az adatkészleten. Lásd: [adja meg az erőforrás-szabályozás](/azure/service-fabric/service-fabric-resource-governance#specify-resource-governance). (Ez a példa a [válaszfal minta](../../patterns/bulkhead.md).)
- Adja meg a processzormagot és memóriát foglaljon le egy szolgáltatáspéldány. További információ a használati és az erőforrás-szabályozási házirendek korlátozásai: [erőforrás-szabályozás](/azure/service-fabric/service-fabric-resource-governance).

Ellenőrizze, hogy minden szolgáltatáspéldány cél vagy replika száma elkerülése érdekében a hibaérzékeny pont (SPOF) 1-nél nagyobb. A legnagyobb számot, amely lehetővé teszi, a szolgáltatáspéldány vagy replika száma az eredménye, hogy, amelyhez a szolgáltatás korlátozza a csomópontok.

Ellenőrizze, hogy minden állapotalapú szolgáltatás legalább két aktív másodlagos replikákon. Öt replikák ajánlottak éles számítási feladatok esetében.

További információkért lásd: [rendelkezésre állását a Service Fabric-szolgáltatások](/azure/service-fabric/service-fabric-availability-services).

## <a name="security-considerations"></a>Biztonsági szempontok

Az alábbiakban az alkalmazás a Service Fabric védelmét biztosító néhány alapvető szempontokat:

### <a name="virtual-network"></a>Virtuális hálózat

Vegye figyelembe, hogy minden egyes virtuálisgép-méretezési csoportba szabályozhatja a kommunikáció áramlását alhálózati határok meghatározása. Mindegyik csomóponttípus külön virtuálisgép-méretezési csoportba a Service Fabric-fürt virtuális hálózaton belül egy alhálózat rendelkezik. Engedélyezi vagy elutasítja a hálózati forgalmat az alhálózatok hálózati biztonsági csoportok (NSG-k) lehet hozzáadni. Például az előtér- és csomóponttípusok, adhat hozzá egy NSG-t csak az előtérbeli alhálózat bejövő forgalmat fogadjon a háttérrendszer alhálózatát.

Ha külső Azure-szolgáltatások hívja meg a fürtről, [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) , ha az Azure-szolgáltatás támogatja ezt a funkciót. A szolgáltatás csak a fürt virtuális hálózati szolgáltatásvégpont használatával védi. Például ha adatok tárolásához a Cosmos DB használ, konfigurálja a Cosmos DB-fiókot, engedélyezze a hozzáférést csak a megadott alhálózat szolgáltatásvégpont. Lásd: [hozzáférés az Azure Cosmos DB-erőforrások virtuális hálózatokról](/azure/cosmos-db/vnet-service-endpoint).

### <a name="endpoints-and-interservice-communication"></a>Végpontok és a szolgáltatások közötti kommunikációt eredményezhet

Ne hozzon létre egy nem biztonságos Service Fabric-fürthöz. A fürt felügyeleti végpontok, a nyilvános internetre tesz közzé, ha a névtelen felhasználók csatlakozhatnak hozzá. Nem biztonságos fürtökhöz nem támogatottak a termelési számítási feladatokhoz. Lásd: [Service Fabric-fürtök biztonsági forgatókönyveit](/azure/service-fabric/service-fabric-cluster-security).

A eredményezhet kommunikáció biztonságossá tételéhez:

- Vegye figyelembe, hogy az ASP.NET Core vagy a Java-webszolgáltatások HTTPS-végpontok engedélyezésére.
- A fordított proxy és a szolgáltatások közötti biztonságos kapcsolat létrehozásához.  További információkért lásd: [csatlakozás a biztonságos service](/azure/service-fabric/service-fabric-reverseproxy-configure-secure-communication).

Ha használ egy [API-átjáró](../../microservices/design/gateway.md), is [hitelesítési kiürítési](../../patterns/gateway-offloading.md) az átjáróhoz. Győződjön meg arról, hogy az egyes szolgáltatások nem érhető el közvetlenül (az API-átjáró) nélkül, kivéve, ha további biztonsági üzenetek hitelesítésére helyen e származnak az átjáróból.

Nem teszik elérhetővé a Service Fabric fordított proxy nyilvánosan. Ekkor minden olyan szolgáltatás, amely kívül a fürt biztonsági rések bemutatása, és ki feleslegesen további információt a fürtön kívülről címezhető lennie HTTP-végpontokat tesznek közzé. Ha azt szeretné, nyilvánosan elérni egy szolgáltatást, használja az API-átjáró. Az egyes beállítások szerepelnek a [API-átjáró](#api-gateway) szakaszban.

A távoli asztal akkor hasznos, ha a diagnosztikai és hibaelhárítási, de győződjön meg arról, ellenkező esetben azt eredményezi, hogy egy biztonsági rés nyitva hagyja, hogy nem.

### <a name="secrets-and-certificates"></a>Titkos kulcsok és tanúsítványok

Engedélyezze a Service Fabric-szolgáltatás eléréséhez a Key Vault titkos kódok, [identitás](/azure/active-directory/managed-identities-azure-resources/services-support-msi#azure-virtual-machine-scale-sets) meg a virtuális gép méretezési csoportot, amely üzemelteti a szolgáltatást. Kódminta: App Service-ből a Key Vault használata a Felügyeltszolgáltatás-identitás segítségével.

Ne használja az ügyféltanúsítványokat a Service Fabric Explorer eléréséhez. Ehelyett használja az Azure Active Directory (Azure AD). Is megtekintheti, [Azure-szolgáltatások, hogy a támogatás az Azure AD-hitelesítés](/azure/active-directory/managed-identities-azure-resources/services-support-msi%23azure-services-that-support-azure-ad-authentication).

Éles környezetben ne használjon önaláírt tanúsítványokat.

A Service Fabric védelmével kapcsolatos további információkért lásd:

- [Az Azure Service Fabric biztonsági áttekintése](/azure/security/azure-service-fabric-security-overview)
- [Az Azure Service Fabric ajánlott biztonsági eljárások](/azure/security/azure-service-fabric-security-best-practices)
- [Az Azure Service Fabric biztonsági ellenőrzőlista](/azure/security/azure-service-fabric-security-checklist)

## <a name="monitoring-considerations"></a>Felügyeleti szempontok

A figyelési lehetőségek előtt javasoljuk, hogy olvassa el ebben a cikkben [diagnosztizálása a Service Fabric gyakori helyzetek](/azure/service-fabric/service-fabric-diagnostics-common-scenarios). Monitorozási adatok a készletekben található felfoghatók:

- [Alkalmazás-metrikák és naplók](#application-metrics-and-logs)
- [A Service Fabric állapotának és az adatok](#service-fabric-health-and-event-data)
- [Infrastruktúra-metrikák és naplók](#infrastructure-metrics-and-logs)
- [Metrikák és naplók függő szolgáltatásokhoz](#dependent-service-metrics)

A két fő lehetőség, hogy az adatelemzés számára az alábbiak:

- Application Insights
- Log Analytics

Használhatja az Azure Monitor irányítópultok figyelés beállításához és az operátorok riasztásokat küldhet. Egyes külső Hálózatfigyelő eszközök, amelyek integrálhatók a Service Fabric, például a dynatrace-szel is vannak. További információkért lásd: [Azure Service Fabric figyelési partnerek](/azure/service-fabric/service-fabric-diagnostics-partners).

### <a name="application-metrics-and-logs"></a>Alkalmazás-metrikák és naplók

Alkalmazáshasználati nyújt a szolgáltatásról, amelyek segítségével adatokat a szolgáltatás állapotának figyelésére, és azonosíthatja a problémákat. A szolgáltatás nyomkövetési és események hozzáadása:

- [Microsoft.Extensions.Logging](/azure/service-fabric/service-fabric-how-to-diagnostics-log#microsoftextensionslogging) a szolgáltatás az ASP.NET Core használatának fejleszt. Más keretrendszerek például Serilog tetszőleges naplózási szalagtár használatára.
- A saját instrumentation használatával adhat hozzá a [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient?view=azure-dotnet) osztályt az SDK-ban, és az adatok megtekintése az Application Insights. Lásd: [egyéni kialakítás hozzáadása az alkalmazáshoz](/azure/service-fabric/service-fabric-tutorial-monitoring-aspnet#add-custom-instrumentation-to-your-application).
- ETW-esemény naplózása használatával [EventSource](/azure/service-fabric/service-fabric-diagnostics-event-generation-app#eventsource). Ez a beállítás alapértelmezés szerint a Visual Studio a Service Fabric megoldás érhető el.

A nyomkövetés és eseménynaplók megtekintéséhez használja [Application Insights](/azure/service-fabric/service-fabric-diagnostics-event-analysis-appinsights) a fogadóként strukturált naplózást, hogy a szolgáltatás nyomkövetési és események jelenítheti meg az egyik. Az Application Insights sok beépített telemetriát nyújt: kérelmek, nyomkövetéseket, események, kivételek, metrikák, függőségeit. A szolgáltatás, Application insights kialakításáról információkért tanulmányozza a következő cikkeket:

- [Oktatóanyag: A Service Fabric Application Insights használatával egy ASP.NET Core alkalmazás monitorozása és diagnosztizálása](/azure/service-fabric/service-fabric-tutorial-monitoring-aspnet).
- [Application Insights for ASP.NET Core](/azure/application-insights/app-insights-asp-net-core)
- [Application Insights .NET SDK](/azure/application-insights/app-insights-api-custom-events-metrics)
- [Az Application Insights SDK-t a Service Fabrichez](https://github.com/Microsoft/ApplicationInsights-ServiceFabric)

ASP.NET Core szolgáltatások használatát a [ILogger felület](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2) az alkalmazásnaplózáshoz. Elérhetővé teszi ezeket az alkalmazásnaplókat az Azure monitorban, küldjön a `ILogger` az eseményeket az Application Insights. További információkért lásd: [az ASP.NET Core alkalmazás ILogger](https://github.com/Microsoft/ApplicationInsights-dotnet-logging/blob/develop/src/ILogger/Readme.md#aspnet-core-application). Az Application Insights tulajdonságokat adhat hozzá korrelációs ILogger események, amelyek akkor hasznos, ha elosztott nyomkövetést megjelenítése.

További információkért lásd:

- [Alkalmazásnaplózás](/azure/service-fabric/service-fabric-diagnostics-event-generation-app)
- [Naplózás hozzáadása Service Fabric-alkalmazáshoz](/azure/azure-monitor/app/correlation)

### <a name="service-fabric-health-and-event-data"></a>A Service Fabric állapotának és az adatok

A Service Fabric telemetriai adatokat is tartalmaz, egészségügyi metrikákkal és eseményekkel kapcsolatban a művelet és a egy Service Fabric-fürt és entitásai teljesítményét: az csomópontok, alkalmazások, szolgáltatások, a partíciók és replikáit.

- [Az EventStore](/azure/service-fabric/service-fabric-diagnostics-eventstore). Egy állapot-nyilvántartó-szolgáltatás, amely a fürt és az entitások kapcsolatos eseményeket gyűjti. A Service Fabric használ az EventStore írása [Service Fabric-események](/azure/service-fabric/service-fabric-diagnostics-event-generation-operational) , adja meg a fürttel kapcsolatos adatokat és állapotának frissítése, hibaelhárítás, figyelése is használható. Azt is összehasonlíthatja a különböző entitások eseményeit, egy adott időben a fürt problémák azonosításához. A szolgáltatás REST API-n keresztül az események közzététele. Az EventStore API-k kapcsolatos információkért lásd: [lekérdezés EventStore API-k a fürthöz kapcsolódó események](/azure/service-fabric/service-fabric-diagnostics-eventstore-query). A Log Analytics az EventStore eseményei úgy konfigurálja a fürt WAD kiterjesztésű tekintheti meg.
- [HealthStore](/azure/service-fabric/service-fabric-health-introduction). A fürt jelenlegi állapotáról pillanatképet tartalmaz. Állapotalapú szolgáltatás, amely összefoglalja az entitások a hierarchia által jelentett összes egészségügyi adatokat. Az adatokat a rendszer vizualizálja [Service Fabric Explorer][sfx]. A HealthStore is figyeli az alkalmazás frissítése. A PowerShell, a .NET-alkalmazás vagy a REST API-k állapotlekérdezések is használhatja. Látható, [Service Fabric állapotmonitorozásának bemutatása](/azure/service-fabric/service-fabric-health-introduction).
- Vegye fontolóra belső egyéni figyelő szolgáltatások. Ezeket a szolgáltatásokat rendszeresen készíthető jelentés az egyéni egészségügyi adatok, például a futó szolgáltatások hibás állapotok. További információkért lásd: [egyéni állapotjelentések](/azure/service-fabric/service-fabric-report-health). A rendszerállapot-jelentések, a Service Fabric explorer használatával olvashatja.  

### <a name="infrastructure-metrics-and-logs"></a>Infrastruktúra-metrikák és naplók

Infrastruktúra-metrikák könnyebben értheti meg a fürtben lévő erőforrás-kiosztást. Az alábbiakban a fő lehetőségeket ezen adatok gyűjtése:

- Windows Azure Diagnostics (WAD). Windows naplók és a csomópont-szintű metrikákat gyűjt. WAD használhatja a IaaSDiagnostics Virtuálisgép-bővítmény konfigurálásával bármely virtuális gép méretezési csoport, amely le van képezve egy csomópont típusa, például a Windows-eseménynaplók, teljesítményszámlálók, ETW/jegyzékek system és a működési eseményeit és egyéni diagnosztikai események gyűjtéséhez. naplók.
- Log Analytics-ügynököket. A Windows biztonságiesemény-naplóinak, a teljesítményszámlálók és az egyéni naplók küldése a Log Analytics MicrosoftMonitoringAgent Virtuálisgép-bővítmény konfigurálása.

Az előző módszerek, például teljesítményszámlálók gyűjtött metrikák típusú átfedés van. Ahol nincs átfedés, javasoljuk a Log Analytics ügynökét használja. Mivel a Log Analytics-ügynök nem az Azure storage, nincs közel valós idejű. Emellett a teljesítményszámlálókat IaaSDiagnostics nem lehet betölteni Log analyticsbe egyszerűen.

![Infrastruktúra megfigyelése a Service Fabricben](./_images/ra-sf-monitoring.png)

A Virtuálisgép-bővítmények használatával kapcsolatos információkért lásd: [Azure virtuális gépi bővítmények és szolgáltatások](/azure/virtual-machines/extensions/overview).

Az adatok megtekintéséhez, konfigurálása a Log Analytics WAD gyűjtött adatok megtekintéséhez. A storage-fiókból olvassa az eseményeket a Log Analytics konfigurálásával kapcsolatos további információkért lásd: [Log Analytics beállítása a fürt](/azure/service-fabric/service-fabric-diagnostics-oms-setup).

Teljesítménynaplók és a Service Fabric-fürt, számítási, hálózati forgalmat, függőben lévő frissítést, és további kapcsolatos telemetriai adatokat is megtekintheti. Lásd: [alkalmazásteljesítmény-figyelés a Log Analytics](/azure/service-fabric/service-fabric-diagnostics-oms-agent).

[A Service Map megoldás a Log Analyticsben](/azure/azure-monitor/insights/service-map) arról nyújt tájékoztatást, a topológia a fürt (azaz futó folyamatokat minden csomópont). Az adatok küldése a storage-fiókban [Application Insights](/azure/monitoring-and-diagnostics/azure-diagnostics-configure-application-insights). Előfordulhat némi késleltetés az adatok Application Insightsban való lekérése közben. Ha meg szeretné tekinteni a az adatok valós idejű, érdemes úgy konfigurálni [Eseményközpont](/azure/monitoring-and-diagnostics/azure-diagnostics-streaming-event-hubs) fogadók és csatornákat. További információkért lásd: [esemény összesítésére és a Windows Azure Diagnostics segítségével gyűjteményt](/azure/service-fabric/service-fabric-diagnostics-event-aggregation-wad).

### <a name="dependent-service-metrics"></a>Függő szolgáltatások mérőszámok

- [Application Insights Alkalmazástérkép](/azure/azure-monitor/app/app-map) lehetővé teszi az alkalmazás topológiájának HTTP függőségi hívások közötti szolgáltatások, a telepített Application Insights SDK használatával.
- [A Service Map megoldás a Log Analyticsben](/azure/azure-monitor/insights/service-map) bejövő és kimenő forgalom/külső szolgáltatásokkal kapcsolatos információkat biztosít. A Service Map ezenkívül integrálható más megoldásokkal, például frissítések vagy biztonsági.
- Egyéni watchdogs hibaállapotok jelentés a külső szolgáltatások segítségével. A szolgáltatás például sikerült jelentést ad health hibajelentésben, ha nem fér hozzá egy külső szolgáltatás vagy az adat storage (az Azure Cosmos DB-hez).  

### <a name="distributed-tracing"></a>Elosztott nyomkövetés

A mikroszolgáltatási architektúra több szolgáltatás gyakran egy adott feladat végrehajtásához részt venni. Az egyes ezeket a szolgáltatásokat a telemetria környezet mező (Műveletazonosító, kérelem azonosítója, és így tovább) egy elosztott nyomkövetést a visszamenőleges korrelációban állnak. Használatával [Alkalmazástérkép](/azure/azure-monitor/app/app-map) az Application Insights, elosztott logikai művelet a nézet létrehozása és az alkalmazás a teljes szolgáltatás grafikon megjelenítése. Tranzakció diagnosztikája az Application Insights kiszolgálóoldali telemetria korrelációját is használhatja. További információkért lásd: [összetevőt tranzakció diagnosztikája egyesített](/azure/application-insights/app-insights-transaction-diagnostics).

[Application Insights Alkalmazástérkép](/azure/azure-monitor/app/app-map) lehetővé teszi az alkalmazás topológiájának HTTP függőségi hívások közötti szolgáltatások, a telepített Application Insights SDK használatával. Fontos a feladatokat, amelyek aszinkron módon üzenetsorral elküldése összekapcsolását is. Az üzenetsori üzenet korrelációs telemetriai adatokat küldenek kapcsolatos részletekért lásd: [instrumentation várólistára](/azure/azure-monitor/app/custom-operations-tracking#queue-instrumentation).

További információkért lásd:

- [A lekérdezés végrehajtása több erőforrás között](/azure/azure-monitor/log-query/cross-workspace-query#performing-a-query-across-multiple-resources)
- [Az Application Insights telemetriai korreláció](/azure/azure-monitor/app/correlation)

#### <a name="alerts-and-dashboards"></a>Riasztások és az irányítópultok

Az Application Insights és a Log Analytics-támogatást egy [részletes lekérdezési nyelvet](/azure/log-analytics/query-language/get-started-queries) (Kusto lekérdezési nyelv), amely lehetővé teszi lekérése és elemzését adatok naplózása. Adathalmazokat hozhat létre, és a diagnosztika irányítópultok megjelenítheti a lekérdezések használatával.

Az Azure Monitor riasztások segítségével részt értesítése, ha bizonyos események történnek az adott erőforrásokhoz. Az értesítés sikerült kell-e-mailek, az Azure-függvényt, egy webhook hívása, és így tovább. További információkért lásd: [az Azure monitorban riasztásokat](/azure/azure-monitor/platform/alerts-overview).

[Keresés naplóriasztási szabályok](/azure/azure-monitor/platform/alerts-unified-log) határozza meg, és rendszeres időközönként Log Analytics-munkaterület Kusto-lekérdezés futtatásához. Egy riasztás akkor jön létre, ha a lekérdezés eredménye megfelel egy bizonyos feltételnek.

## <a name="next-steps"></a>További lépések

- [Tartományelemzés a mikroszolgáltatás-alapú modell használatával](../../microservices/model/domain-analysis.md)
- [A mikroszolgáltatási architektúra tervezése](../../microservices/design/index.md)

[sfx]: /azure/service-fabric/service-fabric-visualizing-your-cluster
