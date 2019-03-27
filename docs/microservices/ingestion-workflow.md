---
title: Adatfeldolgozás és munkafolyamatok a mikroszolgáltatások
description: Adatfeldolgozás és munkafolyamatok a mikroszolgáltatások
author: MikeWasson
ms.date: 10/23/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 75aef5aec7f4663abff45ebdba5dbea245d3ac17
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299500"
---
# <a name="designing-microservices-ingestion-and-workflow"></a>Mikroszolgáltatások tervezése: Adatfeldolgozás és munkafolyamatok

Mikroszolgáltatások gyakran rendelkeznek egy munkafolyamatot egy tranzakció több szolgáltatást is. A munkafolyamat megbízható; kell lennie. nem lehet tranzakció megszakad vagy részlegesen befejezett állapotban hagyja őket. Emellett fontos szabályozni azt a bejövő kérelmek feldolgozási sebességet. Sok kis szolgáltatással kommunikál egymással a bejövő kérések hirtelen kiugrásai túlterhelhetik futó a szolgáltatások közötti kommunikációt eredményezhet.

![A betöltési munkafolyamatának ábrája](./images/ingestion-workflow.png)

> [!NOTE]
> Ez a cikk egy mikroszolgáltatás-alapú referenciaimplementációt nevű alapul a [Drone Delivery alkalmazás](./design/index.md).

## <a name="the-drone-delivery-workflow"></a>A drone delivery munkafolyamat

A Drone Delivery alkalmazást a következő műveleteket kell elvégezni egy kézbesítési ütemezése:

1. Ellenőrizze a felhasználói fiókhoz (szolgáltatásának) állapotát.
2. Hozzon létre egy új csomag entitás (csomag szolgáltatás).
3. Ellenőrzés külső bárminemű a szállítási, szükséges-e a begyűjtés és a továbbítás helyeken alapuló (külső közlekedési szolgáltatás).
4. Ütemezzen egy drónt felvételre (Drónos szolgáltatás).
5. Hozzon létre egy új kézbesítési entitás (Tartalomkézbesítési szolgáltatás).

Ez a központi eleme a teljes alkalmazást, így a teljes körű folyamatot, valamint megbízható, nagy teljesítményű kell lennie. Egyes adott kihívást vonhat:

- **Terheléskiegyenlítés**. Túl sok ügyfél kérést túlterhelhetik eredményezhet hálózati forgalmat a rendszer. Azt is túlterhelhetik háttérrendszer függőségeit, például a storage vagy a távoli szolgáltatások. Ezek a szolgáltatások meghívása őket, a rendszer ellennyomás létrehozása szabályozás előfordulhat, hogy reagálni. Ezért fontos a kérelmeket a rendszer egy puffer, vagy a feldolgozáshoz várólistára helyezésével érkező terhelés.

- **Garantált kézbesítés**. Az Adatbetöltési összetevő bármely ügyfél kérelmeket elkerüléséhez biztosítania kell, legalább egyszeri kézbesítési üzenetek.

- **Hibakezelés**. Ha a szolgáltatások egy hibakódot ad vissza, vagy nem átmeneti hibát tapasztal, a tartalomkézbesítési nem lehet ütemezni. Egy hibakódot jelezheti a várt hibát (például a felhasználói fiók fel van függesztve) vagy egy váratlan kiszolgálóhiba (HTTP 5xx). Elképzelhető, hogy egy szolgáltatás még nem érhető el, a hálózati hívás időtúllépés miatt.

Először áttekintjük az Adatbetöltési oldala egyenlet &mdash; hogyan a rendszer a bejövő felhasználói kérések magasabb átviteli sebességen feldolgozására képes. Azt fogjuk fontolja meg a drone delivery alkalmazás hogyan valósíthat meg egy megbízható munkafolyamatot. Azt tapasztaltuk, hogy az Adatbetöltési alrendszer a kialakítás befolyásolja a munkafolyamat-háttérrendszer.

## <a name="ingestion"></a>Adatbetöltési

Üzleti követelmények alapján, a fejlesztői csapat azonosítja a következő nem funkcionális követelmények támogatunk:

- 10e kérelmek/s folyamatos teljesítményt.
- Tudja kezelni az adatforgalmi csúcsokhoz legfeljebb 50 ezer/mp nélkül ügyfél kérelmeket, vagy túllépik az időkorlátot.
- Kisebb, mint 500ms késés 99 %-a.

A követelmény alkalmanként adatforgalmi kiugrások kezelésére tervezési kihívást mutat be. Elméletileg a rendszer sikerült horizontálisan a maximális várt forgalom kezeléséhez. Azonban kiépítése, hogy számos erőforrás nagyon hatékony. A legtöbb esetben az alkalmazás nem kell, hogy mekkora kapacitást, így lesz tétlen Processzormagok költségszámítási money érték hozzáadása nélkül.

Jobb módszer, hogy a bejövő kérelmek elhelyezi egy puffer, és lehetővé teszik a puffer terheléselosztóként jár el. Ezzel a kialakítással kell, hogy a szolgáltatás tudja kezelni a maximális feldolgozási sebességét rövid időszakokra, de a háttérszolgáltatások csak a maximális fenntartható terhelés kezeléséhez van szükség. Az előtér: pufferelés, a háttérszolgáltatások nem kell nagy adatforgalmi kiugrások kezelésére. A Drone Delivery alkalmazást szükséges méreten [Azure Event Hubs](/azure/event-hubs/) terheléskiegyenlítési esetében megfelelő választás. Az Event Hubs biztosít alacsony késéssel és nagy átviteli sebességet, és a egy költséghatékony megoldás Adatbetöltési nagy mennyiségben.

Tesztelés használtuk egy Standard szintű event hubs átviteli egységeket 100 és 32 partícióval. Megállapítottuk, hogy körülbelül 32 ezer esemény / s betöltési, körül 90ms késéssel. Az alapértelmezett korlát jelenleg 20 átviteli egység, de az Azure-ügyfelek kérhet további átviteli egységek ügyfélszolgálatunknak küldött támogatási kérést. Lásd: [Event Hubs-kvótákról](/azure/event-hubs/event-hubs-quotas) további információt. A metrikák, a számos tényező befolyásolhatja a teljesítményt, például az üzenetek hasznos adatainak mérete, mivel így nem értelmezi őket alapként. További átviteli van szükség, ha a feldolgozó szolgáltatás szegmens egynél több eseményközpont között is. A még nagyobb átviteli sebességet [Event Hubs dedikált](/azure/event-hubs/event-hubs-dedicated-overview) kínál egybérlős üzemelő példánya, amely a bejövő képes kezelni másodpercenként több mint 2 millió esemény.

Fontos megérteni, hogyan érheti el az Event Hubs, az ilyen nagy teljesítményű, mert, amely hatással van, hogy egy ügyfél foglaljanak Event hubs szolgáltatástól érkező üzenetek. Az Event Hubs nem valósít meg egy *várólista*. Ehelyett valósít meg egy *eseménystream*.

Az üzenetsor az egyes fogyasztók távolíthatja el egy üzenetet az üzenetsorból, és a következő fogyasztói az üzenet nem jelenik meg. Üzenetsorok azt ezért lehetővé teszik, hogy egy [versengő felhasználók mintája](../patterns/competing-consumers.md) üzenetek párhuzamos feldolgozását, és a méretezhetőség javítása. A nagyobb rugalmasság érdekében a fogyasztó az üzenet a zárolási tárolja, és a zárolás feloldása, ha az üzenet feldolgozása megtörtént. Ha a feldolgozó meghibásodik &mdash; például a csomóponton futó szoftverleállások &mdash; zárolási túllépi az időkorlátot, és az üzenet vissza alakzatot a várólistára kerül.

![Várólista szemantika ábrája](./images/queue-semantics.png)

Az Event Hubs, másrészről, használja a streamelési szemantikáját. A fogyasztók olvashatja a streamet, egymástól függetlenül saját tempójában. Mindegyik felhasználó az aktuális pozícióját az adatfolyamban nyomon gondoskodik a felelős. A fogyasztó kell írni az aktuális pozícióját adattárolásra néhány előre meghatározott időközönként. Így ha az ügyfél egy tartalék (például a fogyasztói szoftverleállások vagy a gazdagép nem), majd egy új példányt folytathatja a utolsó feljegyzett beosztás érkező adatfolyam olvasása. Ez a folyamat *ellenőrzőpontok*.

Teljesítménybeli megfontolások miatt az olyan fogyasztói általában nem ellenőrzőpont után minden üzenetet. Ehelyett azt rögzített időköz, például a feldolgozás után ad hozzá ellenőrzőpontokat *n* üzeneteket, vagy minden *n* másodperc. Következtében ha egy feldolgozó nem jár sikerrel, néhány esemény előfordulhat, hogy első feldolgozása kétszer, mert egy új példányt mindig szerzi be a legutóbbi ellenőrzőponttól. Nincs a kompromisszummal jár: Gyakori ellenőrzőpontok összeállítása hátrányosan befolyásolhatja a teljesítményt, de a ritka ellenőrzőpontok jelenti azt, további események fog játszani hiba után.

![A stream szemantika ábrája](./images/stream-semantics.png)

Az Event Hubs nem versengő fogyasztó számára készült. Több fogyasztó tudja olvasni a stream, bár egyes is járja a stream egymástól függetlenül. Ehelyett az Event Hubs egy particionált felhasználói mintát használ. Az event hub legfeljebb 32 partícióval rendelkezik. Horizontális skálázás külön fogyasztók rendel mindegyik partíció érhető el.

Ez mit jelent a drone delivery munkafolyamat? A teljes kiaknázásához az Event Hubs lekéréséhez a kézbesítési ütemezési minden üzenet feldolgozása után áthelyezni a következő nem várja. Ebben az esetben, amely azt fogják tölteni legtöbbször a Várakozás a hálózati hívások végrehajtásához. Ehelyett azt kell feldolgoznia az üzenetek aszinkron hívásokat a háttérszolgáltatások használatával párhuzamosan kötegek. Ahogy láthatjuk, a megfelelő ellenőrzőpont-stratégia kiválasztása az is fontos.

## <a name="workflow"></a>Munkafolyamat

Megvizsgáltunk, beolvasása, illetve az üzenetek feldolgozására három lehetőség: Event Processor Host, Service Bus-üzenetsorok és a IoTHub React könyvtárban. Választottuk IoTHub reagálni, de ennek megértéséhez segít az Event Processor Host indítása.

### <a name="event-processor-host"></a>Event Processor Host

Event Processor Host üzenet kötegelés lett tervezve. Az alkalmazás megvalósítja a `IEventProcessor` felületet, és a processzor gazdagép hoz létre egy esemény processzorpéldány minden partíció esetében az eseményközpontban. Az Event Processor Host ekkor meghívja a minden egyes eseményfeldolgozó `ProcessEventsAsync` eseményüzenetek váró metódust. Az ellenőrzőpont belül az Alkalmazásvezérlés a `ProcessEventsAsync` metódust, és az Event Processor Host ellenőrzőpontot ír az Azure storage.

Egy partíción belül Event Processor Host megvárja, amíg `ProcessEventsAsync` vissza úgy, a következő köteg meghívása előtt. Ez a megközelítés leegyszerűsíti a programozási modell, mert a feldolgozás eseménykód nem kell ismételten belépő. Azonban azt is jelenti, hogy az eseményfeldolgozó kezeli egy kötegelt egyszerre, és ez gates a sebesség, amellyel a Processor Host pump is a üzeneteket.

> [!NOTE]
> A processzor gazdagép ténylegesen nem *várjon* abban az értelemben, a szál blokkolása. A `ProcessEventsAsync` metódus aszinkron, ezért az Processor Host végezhet egyéb műveleteket, amíg befejeződik a metódus. De azt nem az adott partíció egy másik üzenetköteget mindaddig, amíg a metódus adja vissza.

A drone alkalmazásban egy üzenetköteget párhuzamosan lehet feldolgozni. De Várakozás a teljes kötegelt végrehajtásához a szűk keresztmetszetet is okozhatnak. Feldolgozási csak lehet olyan gyors kötegelt belül a leglassabb üzenetnek számít. Válaszidők bármilyen változása hozhat létre egy "hosszú tail," ahol néhány lassúak a válaszok húzza le az egész rendszert. A teljesítménytesztek kimutatta, hogy azt nem érte el a célként megadott átviteli sebességet, ezzel a megközelítéssel. Ez a kód *nem* jelenti azt, hogy lehetőleg ne Event Processor Host használatával. A nagy teljesítményű, kerülje bármely hosszú lefutású feladat található, de a `ProcesssEventsAsync` metódust. Az egyes kötegek gyorsan feldolgozni.

### <a name="iothub-react"></a>IotHub React

[IotHub React](https://github.com/Azure/toketi-iothubreact) Akka Streamek könyvtár az Eseményközpontból érkező események olvasását. Akka Streamek egy stream-alapú programozási keretrendszerről, amely megvalósítja a [reaktív Streamek](https://www.reactive-streams.org/) specifikációnak. Hatékony streamelési folyamatok létrehozásával, ahol az összes streamelési műveleteket aszinkron módon történik, és a folyamat szabályosan kezeli ellennyomás hozhat létre egy megoldást kínál. Ellennyomás akkor történik, ha az eseményforrás eredményez, mint az alárendelt fogyasztók fogadhatja gyorsabb ütemben események &mdash; amelynek pontosan az a helyzet akkor, ha a a drone delivery rendszer nevű kategóriáé a forgalom. Ha háttérszolgáltatások lassabban, IoTHub React lelassulnak. Kapacitás nő, ha IoTHub React fogja leküldeni a további üzeneteket, a folyamat keresztül.

Streamelés az Event hubs-Eseményközpontok eseményeinek nagyon természetes programozási modellt Akka Streameket is. Helyett hurkolás események kötegelt keresztül, számos műveletet minden egyes esemény érvényesek, és lehetővé teszik a Akka Streamek kezelni, a streamelési határozza meg. Akka Streamek határozza meg, hogy a streamelési folyamat *források*, *folyamatok*, és *fogadók*. Egy adatforrás generál egy kimeneti adatfolyamba, egy folyamatot egy bemeneti stream feldolgozza, és létrehozza a kimeneti adatfolyamokat és egy fogadó felhasznál egy stream nélkül állít elő kimenetet.

A Scheduler szolgáltatás, amely beállítja a Akka Streamek folyamat a következő kódot:

```java
IoTHub iotHub = new IoTHub();
Source<MessageFromDevice, NotUsed> messages = iotHub.source(options);

messages.map(msg -> DeliveryRequestEventProcessor.parseDeliveryRequest(msg))
        .filter(ad -> ad.getDelivery() != null).via(deliveryProcessor()).to(iotHub.checkpointSink())
        .run(streamMaterializer);
```

Ez a kód az Event Hubs konfigurálja a forrásaként. A `map` utasítás deserializes egyes eseményüzenet be egy Java-osztály, amely egy kézbesítési kérést jelöli. A `filter` utasítás eltávolítja az összes `null` az adatfolyamból objektumokat; ez az eset, amikor egy üzenet nem lehet deszerializálni elleni őröket. A `via` utasítás csatlakoztatja a forrás egy folyamatot, amely minden egyes kézbesítési kérést dolgoz fel. A `to` metódus csatlakoztatja a ellenőrzőpont gyűjtő, amely IoTHub React be van építve a folyamatot.

IoTHub React eseményfeldolgozó állomás, mint más ellenőrzőpontok stratégiát alkalmaz. Ellenőrzőpontok a ellenőrzőpont gyűjtő, amely az a folyamat leállítása szakasz által írt. Akka Streamek kialakítása lehetővé teszi, hogy a folyamat folytatásához a streamelési adatok, miközben a fogadó ír az ellenőrző pont. Ez azt jelenti, hogy a felsőbb rétegbeli feldolgozás szakaszában nem várja meg, megtörténjen ellenőrzőpontok használata szükséges. Konfigurálhatja az ellenőrzőpontok használata után időtúllépés vagy után adott számú üzenetet dolgozott.

A `deliveryProcessor` metódus a Akka Streamek folyamatot hoz létre:

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

A folyamat meghívja a statikus `processDeliveryRequestAsync` metódushoz, amely a tényleges munkát üzenetek feldolgozására.

### <a name="scaling-with-iothub-react"></a>IoTHub reacttel méretezése

A Scheduler szolgáltatás célja, hogy az egyes tárolópéldányok egyetlen partícióról olvas. Például, ha az Event Hubs 32 partícióval rendelkezik, a Feladatütemező szolgáltatás üzembe helyezése 32 replikákkal rendelkező. Ez lehetővé teszi nagy rugalmasságot biztosít a horizontális skálázást tekintetében.

Attól függően, a fürt méretét a fürtben egy csomópont lehet egynél több Scheduler szolgáltatás pod rajta való futtatásához. Azonban ha a Scheduler szolgáltatás több erőforrást igényel, a fürt kiterjeszthető, annak érdekében, hogy a podok szét több csomópontot. A teljesítménytesztek kimutatta, hogy a Scheduler szolgáltatás memória és a hozzászóláslánc-kötött, így teljesítmény nagy mértékben alárendelve, a virtuális gép mérete és a csomópontonként podok számát.

Minden példány tudnia kell, amely az Event Hubs particionálása olvasni. A partíciószám konfigurálásához meggyőződtünk előnye a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) erőforrástípust a Kubernetesben. Egy StatefulSet podok egy állandó azonosítója, amely tartalmaz egy numerikus indexszel rendelkezik. Pontosabban, a pod név `<statefulset name>-<index>`, és ez az érték érhető el a tárolóhoz, a Kubernetes használatával [lefelé irányuló API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/). Futási időben a Scheduler szolgáltatás beolvassa a podnév, és a pod index használja, mint a partícióazonosító.

Ha horizontális felskálázási még tovább a Scheduler szolgáltatás, hozzárendelhet egynél több pod event hub partíciónként, úgy, hogy az egyes partíciók több podok olvas. Azonban ebben az esetben minden példány volna olvasni az események a hozzárendelt partícióban. Ismétlődő feldolgozási elkerülése érdekében kell egy kivonatoló algoritmus használatára, hogy minden példány kihagyja az üzeneteket egy része felett. Ezzel a módszerrel több olvasók használhatnak fel, a streamet, de minden üzenetet csak egy példány dolgoz fel.

![Event hub-kivonatolás ábrája](./images/eventhub-hashing.png)

### <a name="service-bus-queues"></a>Service Bus-üzenetsorok

Egy harmadik lehetőség, hogy mi számít az Event hubs szolgáltatástól érkező üzenetek másolása egy Service Bus-üzenetsorba, és majd a Scheduler szolgáltatás elolvasta az üzenetet a Service Bus. Úgy tűnhet, meglepő írásakor a bejövő kérelmek csak, hogy másolja őket a Service Bus Event hubsba.  Azonban a cél volt, hogy az egyes szolgáltatások különböző előnyeit kihasználva: Az Event Hubs használatával a Service Bus egy versengő felhasználók mintája a számítási feladatok feldolgozásához a várólista szemantikát kihasználva számára a nagy forgalom, az adatforgalmi csúcsokhoz. Ne feledje, hogy a célt a vendégteljesítmény kisebb, mint a várható terhelés, így feldolgozási a Service Bus-üzenetsorba nem kell lennie olyan gyors üzenetbetöltés.

Ezzel a módszerrel proof-of-concept-implementáció érhető el, a 4 KB műveletek száma másodpercenként. Ezeket a teszteket, amelyek bármely valós munkát nem tette meg, de egyszerűen hozzáadott egy rögzített mértékű késés szolgáltatásonként utánzatként funkcionáló háttérszolgáltatások használja. Vegye figyelembe, hogy a teljesítmény-számok is sokkal kevesebb, mint a Service Bus elméleti maximális. Az eltérés okai a következők:

- Nem rendelkezik a különböző ügyfél-paraméterek, például a készlet kapcsolathoz megadott korlátot, a párhuzamos feldolgozás, a lehívott száma és a kötegméret fokú optimális értékeit.

- A hálózati szűk I/O keresztmetszetek.

- Felhasználása [PeekLock](/rest/api/servicebus/peek-lock-message-non-destructive-read) mód helyett [ReceiveAndDelete](/rest/api/servicebus/receive-and-delete-message-destructive-read), amely volt szükség, legalább egyszeri kézbesítési üzenetek biztosítása érdekében.

További teljesítménytesztek előfordulhat, hogy az alapvető ok felderítése és számunkra, hogy a problémák megoldásához. Azonban IotHub React teljesül, a teljesítmény célt, ezért választottuk ezt a lehetőséget. Mindemellett a Service Bus az ebben a forgatókönyvben kivitelezhető lehetőség.

## <a name="handling-failures"></a>Hibák

Nincsenek fontolja meg a hiba három általános osztályba.

1. Előfordulhat, hogy az alárendelt szolgáltatás nem átmeneti hiba, amely minden hiba, amely nem valószínű, hogy megszabadítja önmagában. Nem átmeneti hibák közé tartozik a normál hiba feltételek, például egy érvénytelen bemenet. Is, nem kezelt kivételeket az alkalmazáskód vagy egy folyamat összeomlik. Az ilyen típusú hiba akkor fordul elő, ha a teljes üzleti tranzakció sikertelen kell megjelölni. Ugyanabban a tranzakcióban, amely sikeresen további lépések visszavonása szükség lehet. (A kompenzáló tranzakció, lásd alább.)

2. Egy alárendelt szolgáltatás például egy hálózati időtúllépés átmeneti hibát tapasztalhat. Ezek a hibák gyakran megoldhatók egyszerűen hívása újrapróbálása. Ha a művelet egy bizonyos számú kísérlet után is sikertelen, figyelembe vette nem átmeneti hiba.

3. A Scheduler szolgáltatás előfordulhat, hogy tartalék (például azért, mert összeomlik, egy csomópont). Ebben az esetben a Kubernetes megjelenik a szolgáltatás egy új példányát. Azonban már folyamatban lévő tranzakciók kell folytatni.

## <a name="compensating-transactions"></a>Kompenzáló tranzakciók

Nem átmeneti hiba történik, ha az aktuális tranzakció lehet egy *részben sikertelen* állapot, ahol egy vagy több lépést már sikeresen befejeződött. Például ha a Drone szolgáltatás már ütemezve egy drónt, a drone kell megszakítva. Ebben az esetben az alkalmazásnak kell, hogy sikeres volt-e, a lépések visszavonása egy [kompenzáló tranzakció](../patterns/compensating-transaction.md). Bizonyos esetekben ez kell elvégezni egy külső rendszer, vagy akár egy manuális folyamat.

Ha a kompenzáló tranzakciók logika összetett, érdemes lehet külön szolgáltatás létrehozása felelős a folyamat. A Drone Delivery alkalmazás a Scheduler szolgáltatás sikertelen műveletek be egy dedikált üzenetsorba helyezi. Külön mikroszolgáltatások, a felügyelő nevű ebből az üzenetsorból olvas, és meghívja a lemondás API a meghiúsult lépések kompenzációjához szükséges szolgáltatásokat. Ez a kapcsolat egy változata a [Feladatütemező ügynök felügyeleti mintájának][scheduler-agent-supervisor]. A felügyeleti szolgáltatás előfordulhat, hogy más műveletek is, például szöveg vagy e-mailt a felhasználó értesítése, vagy riasztást küldeni a műveleti irányítópult.

![A felügyelő mikroszolgáltatás bemutató ábra.](./images/supervisor.png)

## <a name="idempotent-versus-non-idempotent-operations"></a>Idempotens, és nem idempotens műveletek

Kérések elveszhetnek, a Feladatütemező szolgáltatás biztosítania kell, hogy minden üzenet legalább egyszer feldolgozása. Az Event Hubs tud garantálni, legalább egyszeri kézbesítési, ha az ügyfél ellenőrzőpontokat megfelelően.

A Scheduler szolgáltatás összeomlik, ha lehet feldolgozni az egy vagy több ügyfél küldött kérelmeket közepén. Ezeket az üzeneteket fog dolgozza fel az ütemező egy másik példánya, és újra fel kell dolgozni. Mi történik, ha a kérés feldolgozása kétszer van? Fontos elkerülhető a munka. Amikor az összes a rendszer két drónok esetében ugyanaz a csomag küldése nem szeretnénk.

Egyik lehetőség, hogy úgy tervezze meg az összes, hogy idempotensek legyenek. Egy művelet idempotens esetén többször hívható az első hívása után további mellékhatásai előállító nélkül. Más szóval egy ügyfél hívhat meg a műveletet egyszer, kétszer, vagy több alkalommal, és az eredmény ugyanaz lesz. A szolgáltatás lényegében, figyelmen kívül hagyja ismétlődő hívásokat. Az, hogy idempotensek lesznek hatásai mód esetén a szolgáltatás képes észlelni a duplikált hívások kell lennie. Például rendelkezhet a hívó hozzárendelése a azonosító ahelyett, hogy a szolgáltatás hozzon létre egy új. A szolgáltatás is tekintse meg az ismétlődő azonosítók.

> [!NOTE]
> A HTTP-specifikációnak megállapítja, hogy a GET, PUT és DELETE-metódusok idempotensnek kell lenniük. POST metódus nem garantált, hogy idempotensek legyenek. Ha egy POST-metódus új erőforrást hoz létre, nincs általánosan garancia arra, hogy-e a művelet idempotens.

Nem mindig idempotens metódus írása könnyen érthető megjegyzésblokkok írására. Egy másik lehetőség, a Scheduler az olyan tartós tárban minden egyes tranzakciót az előrehaladását úgy követheti nyomon. Minden alkalommal, amikor feldolgozza az üzenetet, lenne az állapotát a tartós tárolóban. Minden lépése után azt kellene írni a az eredményt. Előfordulhat, hogy ennek a módszernek a teljesítményre gyakorolt hatása.

## <a name="example-idempotent-operations"></a>Példa: Idempotens műveletek

A HTTP-specifikációnak tájékoztatja, hogy a PUT módszerek idempotensnek kell lenniük. A specifikációnak idempotens ily módon határozza meg:

> A kérelmi metódust "idempotens" számít, ha a tervezett hatása a kiszolgálón a a módszerrel több azonos kérések pedig ugyanaz, mint az egyetlen gyakorolt hatását kérelmet. ([RFC 7231](https://tools.ietf.org/html/rfc7231#section-4))

Új entitás létrehozásakor PUT, POST és szemantika közötti különbségek megértése fontos. Mindkét esetben az ügyfél elküldi az entitás reprezentációját a kérelem törzsében. De értelmében az URI azonosító nem egyezik.

- Egy POST-metódus az URI-t az új entitás, például egy gyűjteményt egy szülőerőforrás jelenti. Például hozzon létre egy új szállítási, hogy az URI-t lehet `/api/deliveries`. A kiszolgáló hoz létre az entitást, és hozzárendeli egy új URI-t, mint például `/api/deliveries/39660`. Ez az URI a válasz Location fejléce adja vissza. Minden alkalommal, amikor az ügyfél elküld egy kérelmet, a kiszolgáló hoz létre egy új entitást egy új URI-t.

- Az URI-t egy PUT metódust a azonosítja az entitást. Ha már van egy entitás az URI-ra, a kiszolgáló felülírja a létező entitásba a változattal a kérésben. Ha nem entitás létezik, az URI-ra, a kiszolgáló létrehoz egyet. Tegyük fel, az ügyfél küld egy PUT kérelem a `api/deliveries/39660`. Feltételezve, hogy nincs kézbesítési az URI-ra, a kiszolgáló létrehoz egy újat. Most már az ügyfél elküld a kérésben újra, ha a kiszolgáló felülírja a létező entitásba.

Íme a PUT metódust a kézbesítési szolgáltatás megvalósítását.

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

Valószínű, hogy a legtöbb kérelmek létrehoz egy új entitást, így optimistically metódushívások `CreateAsync` a tárház objektum majd kezeli az olyan ismétlődő-erőforrás kivételek, amelyek inkább az erőforrás frissítésével.

<!-- links -->

[scheduler-agent-supervisor]: ../patterns/scheduler-agent-supervisor.md