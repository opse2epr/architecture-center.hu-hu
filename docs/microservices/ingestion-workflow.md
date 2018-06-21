---
title: Adatfeldolgozást és mikroszolgáltatások munkafolyamata
description: Adatfeldolgozást és mikroszolgáltatások munkafolyamata
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 6477c3f2b0cc6d37dcd4637dc0dde4f7a6e3cc74
ms.sourcegitcommit: 94c769abc3d37d4922135ec348b5da1f4bbcaa0a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/13/2017
ms.locfileid: "26678730"
---
# <a name="designing-microservices-ingestion-and-workflow"></a>Mikroszolgáltatások tervezése: adatfeldolgozást és a munkafolyamat

Mikroszolgáltatások gyakran olyan munkafolyamatot, amely egyetlen tranzakció több szolgáltatáshoz is rendelkezik. A munkafolyamat megbízható; kell lennie. nem lehet tranzakciók elveszíti vagy részben befejezett állapotban hagyja őket. Nagyon fontos is a bejövő kérelmek adatfeldolgozást átviteli sebességét. A sok kisméretű szolgáltatások kommunikál egymással a bejövő kérelmek kapacitásnövelés is ne terhelje tovább a értekezleteire kommunikációt. 

![](./images/ingestion-workflow.png)

## <a name="the-drone-delivery-workflow"></a>A dron kézbesítési munkafolyamatot

A dron továbbítási alkalmazást hozhat létre a következő műveleteket kell elvégezni egy kézbesítési ütemezése:

1. Tekintse meg a felhasználói fiókhoz (fiókszolgáltatás).
2. Hozzon létre egy új csomag entitás (csomag szolgáltatás).
3. Jelölőnégyzet külső szállítására a kézbesítéshez szükséges-e a felvételi és kézbesítési helyeken alapuló (külső szállítására szolgáltatás).
4. Ütemezhet egy dron felvételhez (dron szolgáltatás).
5. Hozzon létre egy új kézbesítési entitás (kézbesítési szolgáltatás).

Ez az a teljes alkalmazás alapszolgáltatásai, ezért a végpont folyamat, valamint megbízható performant kell lennie. Egyes adott kihívásokkal kell figyelembe venni:

- **Terheléskiegyenlítés**. Túl sok ügyfél kérést is ne terhelje tovább a rendszer értekezleteire hálózati forgalmat. Azt is is ne terhelje tovább háttér függőségek, például a tároló- és távoli szolgáltatások. Ezek a szolgáltatások hívja, ellennyomás létrehozásakor a rendszer szabályozásával előfordulhat, hogy reagálni. Ezért fontos a kérelmeket a rendszer által a üzembe egy puffer vagy feldolgozásra várólista beérkező terhelés. 

- **Garantált kézbesítés**. Bármely ügyfél kérelmeket elkerüléséhez az adatfeldolgozást összetevő kell garantálja, legalább egyszeri üzenetek. 

- **Hibakezelés**. Ha a szolgáltatások egy hibakódot ad vissza, vagy nem átmeneti hibát tapasztal, a kézbesítés nem lehet ütemezni. Hibakód utalhat a várható hibaállapotot (például a felhasználói fiók fel van függesztve) vagy egy váratlan kiszolgálóhiba (HTTP 5xx). Elképzelhető, hogy a szolgáltatás is érhető el, amely a hálózati hívás túllépi az időkorlátot. 

Először megnézzük a egyenlet adatfeldolgozást oldalán &mdash; hogyan a rendszer magas teljesítmény bejövő felhasználói kérelmek fogadására képes. Azt fogja fontolja meg a dron továbbítási alkalmazást hozhat létre hogyan valósíthatja meg a megbízható munkafolyamat. Változik, hogy a kialakítás az adatfeldolgozást alrendszer befolyásolja a munkafolyamat háttér. 

## <a name="ingestion"></a>Adatfeldolgozást

Üzleti követelmények alapján, a fejlesztői csapat meghatározott szempontjából a következő művelet követelmények:

- 10e kérelmek/másodperc fenntartható átviteli sebességet.
- Tudja kezelni a teljesítményt, legfeljebb 50K másodpercenkénti nélkül ügyfél kérelmeket, vagy időtúllépés miatt.
- Kisebb, mint 500ms várakozási ideje a a 99th PERCENTILIS visszaadása.

A követelmény alkalmi igényeiben jelentkező forgalmának kezeléséhez tervezési kihívást mutatja be. Elméletileg a rendszer sikerült horizontálisan a maximális várt forgalom kezeléséhez. Azonban kiépítése, hogy számos olyan forrás nagyon hatékony. Az esetek többségében az alkalmazást nem kell, hogy mekkora kapacitást, nem lenne üresjárati mag, költségszámítás money érték hozzáadása nélkül.

A jobb megoldás, a bejövő kéréseket kerüljenek a puffert, és lehetővé teszik a terhelés leveler összekötőként puffer. Ezzel a kialakítással a adatfeldolgozást szolgáltatás képes kezelni a maximális adatfeldolgozást sebességét rövid időszakokra kell lennie, de a háttér-szolgáltatások csak kell a maximális fenntartható terhelés kezelésére. Pufferelés első végén, a háttér-szolgáltatások nem kell kezelni a forgalom nagy teljesítményt. A dron továbbítási alkalmazást hozhat létre, a szükséges léptékű [Azure Event Hubs](/azure/event-hubs/) a terhelés simítás jó választás. Az Event Hubs alacsony késéssel és nagy teljesítményt biztosít, és egy költséghatékony megoldást a következő magas adatfeldolgozást kötetek. 

A tesztelés során a egy Standard csomagra eseményközpont 32 partíciók és 100 átviteli egységek használtuk. Megfigyelhető, hogy hamarosan 32 KB-os esemény / másodperc adatfeldolgozást, késéssel 90ms körül. Jelenleg az alapértelmezett érték 20 átviteli egység, de az Azure-ügyfél kérhetnek további átviteli egységek tervátalakítási egy támogatási kérést. Lásd: [Event Hubs kvóták](/azure/event-hubs/event-hubs-quotas) további információt. Csakúgy, mint a metrikák sok tényezők befolyásolhatják a teljesítményt, például az üzenet terhelés méretének, így nem értelmezi őket egy alapként. További átviteli van szükség, ha az adatfeldolgozást szolgáltatás shard egynél több eseményközpont között is. A még nagyobb átviteli sebességet [Event Hubs dedikált](/azure/event-hubs/event-hubs-dedicated-overview) biztosít egy bérlői központi telepítéseket is érkező másodpercenként több mint 2 millió esemény.

Fontos megérteni, hogyan Event Hubs ilyen magas teljesítmény érhető el, mert, amely hatással van, hogyan ügyfél foglaljanak Eseményközpontokból származó üzenetek. Az Event Hubs nem valósítja meg a *várólista*. Ahelyett, hogy, amely egy *eseményfelhasználó*. 

A várólistához egy egyedi felhasználói eltávolítása egy üzenetet az üzenetsorból, és a következő fogyasztói az üzenet nem jelenik. Várólisták ezért használatát teszik lehetővé a [versengő fogyasztók mintát](../patterns/competing-consumers.md) párhuzamosan üzenetek feldolgozásához, és a méretezhetőség javítása. Nagyobb rugalmasság a fogyasztó tárolja a zárolást az üzenet és zárolás feloldása, amikor az üzenet feldolgozása megtörtént. Ha a fogyasztó nem sikerül &mdash; például a csomóponton futó összeomlások &mdash; a zárolás végrehajtásának időkorlátja, és az üzenet vissza a várólista-kiszolgálóra kerül. 

![](./images/queue-semantics.png)

Az Event Hubs, másrészt adatfolyam szemantikáját használja. A fogyasztók olvashatja a streamet egymástól függetlenül saját tempójában. Mindegyik felhasználó felelős nyomon követése céljából az aktuális pozíciót az adatfolyamban. A fogyasztó az egyes előre meghatározott időközönként állandó tároló kell írni a jelenlegi állapotában. Így ha a fogyasztó (például a fogyasztói összeomlik vagy az állomás nem tud) hibát észlel, majd új példányt folytathatja az adatfolyam olvasásakor az utolsó rögzített hely. Ez a folyamat *ellenőrzőpontok*. 

A megfelelő teljesítmény érdekében egy végfelhasználói általában nem ellenőrzőpont után minden üzenetet. Ehelyett az egyes rögzített időközönként, például a feldolgozást követően az ellenőrzőpontok  *n*  üzeneteket, vagy minden  *n*  másodperc. Következésképpen a fogyasztó nem sikerül, ha egyes események előfordulhat, hogy feldolgozni kétszer, mert egy új példány mindig szerzi be a legutóbbi ellenőrzőponttól. Nincs olyan kompromisszumot: gyakori ellenőrzőpontokat hátrányosan befolyásolhatja a teljesítményt, de a ritka ellenőrzőpontokat jelenti azt, egy meghibásodás után fog visszajátszásos a további események.  

![](./images/stream-semantics.png)
 
Az Event Hubs nem versengő fogyasztó számára készült. Bár több felhasználóból adatfolyam tudja olvasni, minden egyes halad át a adatfolyam egymástól függetlenül. Ehelyett az Event Hubs egy particionált felhasználói mintát használ. Az eseményközpontok legfeljebb 32-partíciókkal rendelkezik. Horizontális skálázhatóságot az érhető el egy külön végfelhasználói hozzárendelése minden partíció.

Ez mit jelent a dron kézbesítési munkafolyamat? Ahhoz, hogy a teljes előnye, hogy az Event Hubs, a kézbesítési ütemezési nem Várjon, amíg minden üzenetet dolgozható fel, mielőtt a Tovább gombra. Ha mégis meghaladja, az idő, Várakozás a hálózati hívás befejezéséhez fog legmagasabbak. Ehelyett kell kötegek párhuzamosan, a háttér-szolgáltatásokhoz való aszinkron hívásokkal üzenetek feldolgozásához. Megtanulhatja, mivel a megfelelő ellenőrzőpont-stratégia kiválasztása az is fontos.  

## <a name="workflow"></a>Munkafolyamat

Olvasási és az üzenetek feldolgozása három lehetséges nézett azt: Event Processor Host, Service Bus-üzenetsorok és az IOT hubbal reagálni könyvtár. IOT hubbal reagálni választottuk, de megértéséhez Event Processor Host kezdődnie segíti. 

### <a name="event-processor-host"></a>Event Processor Host

Event Processor Host készült üzenet kötegelés. Az alkalmazás megvalósítja a `IEventProcessor` felületet, és a Processor Host létrehoz egy esemény processzorpéldány minden partíció esetében az eseményközpontba. Az Event Processor Host majd meghívja a minden esemény processzor `ProcessEventsAsync` eseményüzeneteket kötegekben metódust. Az alkalmazás vezérlők az ellenőrzőpont belül a `ProcessEventsAsync` metódust, és az Event Processor Host ellenőrzőpontot ír az Azure storage. 

A partíciókon belül Event Processor Host megvárja-e a `ProcessEventsAsync` vissza a következő mérési adatköteget való ismételt hívása előtt. Ezt a módszert használja egyszerűbbé teszi a programozási modellel, mert a feldolgozás eseménykód nem szükséges ismételten belépő. Azonban azt is jelenti, hogy a eseményfeldolgozóhoz kezeli egy kötegelt egyszerre, és ez a sebesség, amellyel a Processor Host üzenetek is szivattyú gates.

> [!NOTE] 
> A Processor Host ténylegesen nem *Várjon, amíg* abban az értelemben az egy szál blokkolás. A `ProcessEventsAsync` metódus aszinkron, ezért a Processor Host teheti más feladatok, amíg befejeződik az metódus. De azt nem az adott partíció üzenetek egy másik köteg mindaddig, amíg a metódus visszaadja. 

A dron alkalmazásban az üzenetkötegek párhuzamosan lehet feldolgozni. De a teljes kötegelt befejezéséhez Várakozás a szűk keresztmetszetek is okozhatnak. Feldolgozási csak lehet gyorsaságának a leglassabb üzenet egy kötegben. Válaszidők bármilyen változása a "hosszú utóhívás," ahol néhány lassú válaszok húzza le az egész rendszert hozhat létre. A teljesítménytesztek bemutatta, hogy azt nem érte el az ezzel a megközelítéssel cél átviteli sebességet. Ez a does *nem* jelenti azt, hogy Event Processor Host használata kerülendő. A magas teljesítmény kerülje a hosszan futó feladatokat belül, de a `ProcesssEventsAsync` metódust. Gyors feldolgozása egyes kötegekben.

### <a name="iothub-react"></a>Reagálni IOT hubbal 

[IOT hubbal reagálni](https://github.com/Azure/toketi-iothubreact) események olvasása az Eseményközpont Akka adatfolyamok könyvtár. Akka adatfolyamok egy adatfolyam-alapú programozási keretrendszerről, amely megvalósítja a [reaktív adatfolyamok](http://www.reactive-streams.org/) megadását. Is biztosítja az adatfolyam-továbbítási hatékony folyamatok, ahol minden adatfolyam műveleteket aszinkron módon történik, és az adatcsatorna szabályosan kezeli ellennyomás létrehozásához. Ellennyomás akkor fordul elő, amikor egy eseményforrás eredményez, mint az alárendelt fogyasztók fogadhatja gyorsabban események &mdash; amelyhez pontosan a helyzet akkor, ha a a dron kézbesítési rendszer rendelkezik a csúcs a forgalom. Ha a háttérkiszolgáló szolgáltatások lassabban go, IOT hubbal reagálni lelassulnak. Ha kapacitás nő, IOT hubbal reagálni fogja leküldeni a több üzenet a feldolgozási folyamaton keresztül.

Az Event Hubs eseményeit streaming nagyon természetes programozási modellt Akka adatfolyamokat is. Helyett egy kötegelt események ismétlése, határozza meg, amely minden esemény alkalmazandó, és tájékoztassa kezelni a streaming Akka adatfolyamok műveletkészlet. Akka adatfolyamok határozza meg a folyamatos átviteli folyamat *források*, *Forgalomáramlás*, és *fogadók esetében*. Forrás hoz létre egy kimeneti adatfolyamba, a folyamat egy bemeneti adatfolyam feldolgozza, és hozza létre a kimeneti adatfolyamokat, és a fogadó fel adatfolyam nélkül állít elő kimenetet.

A kód itt látható a Feladatütemező szolgáltatás, amely beállítja a Akka adatfolyamok folyamat:

```java
IoTHub iotHub = new IoTHub();
Source<MessageFromDevice, NotUsed> messages = iotHub.source(options);

messages.map(msg -> DeliveryRequestEventProcessor.parseDeliveryRequest(msg))
        .filter(ad -> ad.getDelivery() != null).via(deliveryProcessor()).to(iotHub.checkpointSink())
        .run(streamMaterializer);
```

Ezt a kódot állít be az Event Hubs. A `map` utasítás minden eseményüzenet deserializes egy Java-osztály egy kézbesítési kérelem jelölő be. A `filter` utasítás eltávolítja minden `null` be az adatfolyamból objektumok; ebben az esetben, ha egy üzenet nem deszerializálható megóvja. A `via` utasítás csatlakoztatja a forrás egy folyamatot, amely minden kézbesítési kérést dolgoz fel. A `to` metódus az ellenőrzőpont fogadó, IOT hubbal reagálni rendszerbe beépített csatlakoztatja a folyamatot.

IOT hubbal reagálni használja egy másik ellenőrzőpontok stratégia esemény Host processzor-nál. Az ellenőrzőpontok a ellenőrzőpont fogadó, amely a lezáró szakasza a folyamatnak írja. Akka adatfolyamokat a kialakítása lehetővé teszi, hogy a folytatáshoz a streamelési adatok, miközben a fogadó ír az ellenőrzőpont-feldolgozási folyamat. Ez azt jelenti, hogy a felsőbb rétegbeli feldolgozás szakaszában nem kell várnia az ellenőrzőpontok használatával fordulhat elő. Konfigurálhatja az ellenőrzőpontok létrehozása után egy időtúllépési, vagy egy bizonyos számú üzenetek feldolgozása után.

A `deliveryProcessor` hoz létre a Akka adatfolyamok folyamata:  

```java
private static Flow<AkkaDelivery, MessageFromDevice, NotUsed> deliveryProcessor() {
    return Flow.of(AkkaDelivery.class).map(delivery -> {
        CompletableFuture<DeliverySchedule> completableSchedule = DeliveryRequestEventProcessor
                .processDeliveryRequestAsync(delivery.getDelivery(), 
                        delivery.getMessageFromDevice().properties());
        
        completableSchedule.whenComplete((deliverySchedule,error) -> {
            if (error!=null){
                Log.info("failed delivery" + error.getStackTrace());
            }
            else{
                Log.info("Completed Delivery",deliverySchedule.toString());
            }
                                
        });
        completableSchedule = null;
        return delivery.getMessageFromDevice();
    });
}
```

A folyamat meghívja a statikus `processDeliveryRequestAsync` módszer, amely a minden üzenet feldolgozásakor a tényleges munkát.

### <a name="scaling-with-iothub-react"></a>Az IOT hubbal reagálni skálázás

A Feladatütemező szolgáltatás célja, hogy a tároló feltünteti olvassa be az egyetlen partícióra. Például, ha az Event Hubs 32-partíciókkal rendelkezik, a Feladatütemező szolgáltatás telepítése 32 replikával. Ez lehetővé teszi a magas fokú rugalmasságot biztosít horizontális skálázás tekintetében. 

Attól függően, hogy a fürt méretét a fürt egyik csomópontjában levő rendelkezhet egynél több Feladatütemező szolgáltatás pod fut rajta. De ha a Feladatütemező szolgáltatás több erőforrást igényel, a fürt kiterjeszthető, ahhoz, hogy a három munkaállomás-csoporttal szét több csomópontot. A teljesítménytesztek bemutatta, hogy a Feladatütemező szolgáltatás határa memória - és szál-, így teljesítmény alárendelve nagy mértékben a Virtuálisgép-méretet és az egyes csomópontok három munkaállomás-csoporttal.

Minden példány tudnia kell, amely az Event Hubs partícióazonosító olvasni. Konfigurálhatja a számát, azt előnyt a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) Kubernetes erőforrástípus. Az egy StatefulSet három munkaállomás-csoporttal rendelkezik, amely tartalmazza az tartozó numerikus index állandó azonosítóval. Pontosabban, a pod értéke `<statefulset name>-<index>`, és ezt az értéket a tárolóhoz a Kubernetes keresztül érhető el [lefelé API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/). Futásidőben a Feladatütemező szolgáltatás a pod felhasználónevét olvassa be, és a pod index használja, mint a partícióazonosító.

Horizontális a Feladatütemező szolgáltatás még tovább szükség esetén sikerült event hub partíciónként több pod rendelhet, hogy több három munkaállomás-csoporttal mindegyik partíció olvas. Azonban ebben az esetben minden példány volna olvasni az események a hozzárendelt partícióban. Ismétlődő feldolgozásának elkerülése érdekében meg kell a kivonatolási algoritmust használja, hogy az egyes példányok kihagyja az üzenetek egy része felett. Ezzel a módszerrel több olvasók felhasználhat az adatfolyam, de minden üzenetet csak egy példány dolgoz fel. 
 
![](./images/eventhub-hashing.png)

### <a name="service-bus-queues"></a>Service Bus-üzenetsorok

Egy harmadik beállítás, amely azt tekinti az üzenetek átmásolni az Event Hubs egy Service Bus-üzenetsorba, és majd a Feladatütemező szolgáltatás elolvasta az üzeneteket a Service Bus. A bejövő kéréseket írása az csak a másolja őket a Service Bus Event Hubsban furcsa tűnhet.  Azonban az ötletére lett-e használni a különböző szintjeiről minden egyes szolgáltatás: használja az Event Hubs nagy forgalom igényeiben jelentkező befogadására közben a várólista szemantikáját a Service Bus számára, a munkaterhelés versengő fogyasztók mintával feldolgozni. Ne feledje, hogy a cél a folyamatos átviteli kisebb, mint a várható terhelést, így feldolgozásra a Service Bus-üzenetsorba nem kell lennie az üzenet olyan gyors adatfeldolgozást.
 
Ezt a módszert használja az a koncepció igazolása megvalósítási érhető el, 4 KB-os műveletek másodpercenkénti száma. Ezeket a teszteket, amelyek még nem tette tényleges munka, de egyszerűen hozzáadni a szolgáltatás várakozási ideje a rögzített méretű utánzatait háttér szolgáltatások használt. Vegye figyelembe, hogy a teljesítmény számok volt sokkal kisebb, mint a Service Bus elméleti maximum. Az eltérés okai a következők:

- Nem rendelkezik a különböző ügyfél-paraméterek, például a kapcsolat készlet kapacitása, párhuzamos folyamatkezelést biztosítja, a lehívott, valamint a Köteg mérete fokának optimális értékeit.

- A hálózati i/o-szűk keresztmetszeteket.

- Használja a [PeekLock](/rest/api/servicebus/peek-lock-message-non-destructive-read) mód helyett [ReceiveAndDelete](/rest/api/servicebus/receive-and-delete-message-destructive-read), amely volt szükség, annak érdekében, legalább egyszeri üzenetek kézbesítését.

További teljesítménytesztek előfordulhat, hogy az alapvető ok észlelt és engedélyezett a problémák megoldásához. Azonban IOT hubbal reagálni teljesül, a teljesítmény cél, válassza ezt a beállítást. Említett, a Service Bus ehhez a forgatókönyvhöz kivitelezhető lehetőség.

## <a name="handling-failures"></a>Hibák kezelése 

Nincsenek három általános osztályok nem kell figyelembe venni.

1. Előfordulhat, hogy az alárendelt szolgáltatás nem átmeneti hiba, amely minden, amely nem valószínű, hogy eltűnik önmagában hiba. Nem tranziens hibák normál hibaállapotokat, például egy metódusnak érvénytelen a megadott tartalmazza. Is, nem kezelt kivételeket az alkalmazáskód vagy egy folyamat összeomló. Ilyen hiba akkor fordul elő, ha a teljes üzleti tranzakció hibaként kell megjelölni. Elképzelhető, hogy más lépéseket ugyanabban a tranzakcióban már sikeresen telepített szükségessé. (Kompenzációs tranzakciók lásd alább.)
 
2. Az alárendelt szolgáltatás problémákat tapasztalhat a például a hálózati időtúllépés átmeneti hiba történt. Ezek a hibák gyakran megoldhatók egyszerűen újra próbálkozik a hívást. Ha egy bizonyos számú kísérlet után továbbra is sikertelen a művelet, nem átmeneti hiba figyelembe. 

3. A Feladatütemező szolgáltatás előfordulhat, hogy fault (például azért, mert a csomópont összeomlik). Ebben az esetben Kubernetes megjelenik a szolgáltatás egy új példányát. Már folyamatban lévő tranzakciók, kell folytatni. 

## <a name="compensating-transactions"></a>Kompenzációs tranzakciók

Nem átmeneti hiba akkor fordul elő, ha az aktuális tranzakció lehet egy *részben sikertelen* állapotba, ahol egy vagy több lépése már sikeresen befejeződött. Például ha a dron szolgáltatás már ütemezve egy dron, a dron kell szakítható meg. Ebben az esetben az alkalmazást kell visszavonja a lépéseket, a sikeresen telepített egy [Compensating tranzakció](../patterns/compensating-transaction.md). Bizonyos esetekben kell erre a külső rendszerek vagy akár egy kézi művelet szerint. 

Ha tranzakciók kompenzációs logika összetett, érdemes lehet külön szolgáltatás létrehozása a folyamat felelős. Az a dron továbbítási alkalmazást hozhat létre a Feladatütemező szolgáltatás egy dedikált várólista, a sikertelen műveleteket helyezi. Egy külön mikroszolgáltatási, a felügyelő nevű olvassa be ebből a várólistából, és meghívja a API törlése a szolgáltatásokat, amelyeket a helyesbítéshez. Ez a módosítás a [Feladatütemező ügynök felügyelő mintát][scheduler-agent-supervisor]. A felügyeleti szolgáltatás eltarthat más műveleteket is, például szöveg vagy e-mail a felhasználó értesítése, vagy egy riasztást küldjön egy műveletek irányítópultot. 

![](./images/supervisor.png)

## <a name="idempotent-vs-non-idempotent-operations"></a>Az Idempotent vs az idempotent műveletek

Minden kérést elveszhetnek, a Feladatütemező szolgáltatás biztosítania kell, hogy minden üzenet legalább egyszer feldolgozása. Az Event Hubs képes garantálni, legalább egyszeri kézbesítési, ha az ügyfél ellenőrzőpontokat megfelelően.

A Feladatütemező szolgáltatás összeomlik, ha egy vagy több ügyfél kérelmeket feldolgozó közepén lehet. Azokat az üzeneteket a rendszer az ütemező egy másik példánya észlelnie, majd újra fel kell dolgozni. Mi történik, ha a kérés feldolgozása kétszer van? Fontos munka duplikálását elkerülje. Amikor az összes a rendszer két dronok az ugyanazon csomag küldése nem szeretnénk.

Egyik módszer az, hogy az idempotent összes műveletek. Egy műveletet az idempotent esetén többször hívható az anélkül további mellékhatásokkal első hívása után. Más szóval ügyfél hívhat meg a műveletet egyszer, kétszer, vagy többször, és az eredmény ugyanaz lesz. Alapvetően a szolgáltatás figyelmen kívül hagyja-ismétlődő hívások. A mellékhatással működő kell lennie az idempotent mód esetén a szolgáltatás képes észlelni az ismétlődő hívások kell lennie. Például lehet a hívó hozzárendelés azonosítója ahelyett, hogy a szolgáltatás létrehozása egy új. A szolgáltatás majd ellenőrizheti az ismétlődő azonosító.

> [!NOTE]
> A HTTP-specifikáció szerint, hogy GET, PUT és DELETE módszerek idempotent kell lennie. POST metódus nem garantált, hogy az idempotent lehet. Ha a POST metódussal hoz létre egy új erőforrást, akkor általában nem garantálja, hogy a művelet nem idempotent. 

Nincs mindig egyszerű idempotent metódus írása. Lehetősége van az ütemezőt, hogy minden tranzakció tartós tárolójában előrehaladását úgy követheti nyomon. Amikor egy üzenetet feldolgozza, lenne, a tartós tárolójában állapotát. Minden lépés után azt kellene írni a eredménye. Ez a megközelítés a teljesítményre gyakorolt hatása lehet.

## <a name="example-idempotent-operations"></a>Példa: Az Idempotent műveletek

A HTTP-specifikáció szerint PUT módszerek idempotent kell lennie. A meghatározás meghatározza, hogy az idempotent ily módon:

>  A kérési metódust akkor tekinthető "idempotent", ha a tervezett hatása a kiszolgálón, ez a módszer több azonos kérelmek megegyezik a hatás egyetlen ilyen kérelem. ([RFC 7231](https://tools.ietf.org/html/rfc7231#section-4))

Fontos PUT és a FELADÁS egy vagy több szemantikájú új entitás létrehozásakor közötti különbségek megértése. Mindkét esetben az ügyfél elküldi egy entitást képviselő alakot a kérés törzsében. Az URI azonosító szerinti azonban különböző.

- A POST metódus az URI új entitás, például egy gyűjtemény szülő erőforrását jelöli. Például hozzon létre egy új kézbesítési, az URI azonosító lehet `/api/deliveries`. A kiszolgáló az entitások, és hozzárendeli egy új URI, például a `/api/deliveries/39660`. Ezt az URI eredmény abban az esetben a helyet megjelölő fejlécet a válasz. Minden alkalommal, amikor az ügyfél elküldi a kérelmet, a kiszolgáló hoz létre egy új entitást egy új URI.

- Egy PUT metódust az URI azonosítja az entitást. Már létezik egy entitás, hogy az URI-azonosítóhoz, ha a kiszolgáló lecseréli a meglévő entitás a verziót a kérelemben. Ha adott URI nem entitás létezik, a kiszolgáló létrehoz egy. Tegyük fel az ügyfél elküldi a PUT kérelmek `api/deliveries/39660`. Feltételezve, hogy nincs kézbesítését, hogy az URI-azonosítójú, a kiszolgáló létrehoz egy újat. Miután az ügyfél elküldi a kérésben újra, ha a kiszolgáló lecseréli a meglévő entitás.

Ez a kézbesítési szolgáltatás végrehajtása a PUT metódust. 

```csharp
[HttpPut("{id}")]
[ProducesResponseType(typeof(Delivery), 201)]
[ProducesResponseType(typeof(void), 204)]
public async Task<IActionResult> Put([FromBody]Delivery delivery, string id)
{
    logger.LogInformation("In Put action with delivery {Id}: {@DeliveryInfo}", id, delivery.ToLogInfo());
    try
    {
        var internalDelivery = delivery.ToInternal();

        // Create the new delivery entity.
        await deliveryRepository.CreateAsync(internalDelivery);

        // Create a delivery status event.
        var deliveryStatusEvent = new DeliveryStatusEvent { DeliveryId = delivery.Id, Stage = DeliveryEventType.Created };
        await deliveryStatusEventRepository.AddAsync(deliveryStatusEvent);

        // Return HTTP 201 (Created)
        return CreatedAtRoute("GetDelivery", new { id= delivery.Id }, delivery);
    }
    catch (DuplicateResourceException)
    {
        // This method is mainly used to create deliveries. If the delivery already exists then update it.
        logger.LogInformation("Updating resource with delivery id: {DeliveryId}", id);

        var internalDelivery = delivery.ToInternal();
        await deliveryRepository.UpdateAsync(id, internalDelivery);

        // Return HTTP 204 (No Content)
        return NoContent();
    }
}
```

Várható, hogy legtöbb kérelem létrehoz egy új entitást, így optimistically metódushívások `CreateAsync` a tárház objektum, majd kezeli az erőforrás helyette frissítésével duplikált-resource kivételei. 

> [!div class="nextstepaction"]
> [API-átjárókkal](./gateway.md)

<!-- links -->

[scheduler-agent-supervisor]: ../patterns/scheduler-agent-supervisor.md