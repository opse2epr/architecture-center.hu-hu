---
title: Jogcím-minta
titleSuffix: Cloud Design Patterns
description: Nagy méretű üzenet felosztása egy jogcím-ellenőrzést és a egy hasznos egy üzenetbusz túlterhelésének elkerülése érdekében.
keywords: tervezési minta
author: yorek
ms.date: 03/05/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 88457983df4ea3fd4ed8d0c447de6302422339f8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299545"
---
# <a name="claim-check-pattern"></a>Claim-Check Pattern

Nagy méretű üzenet felosztása egy jogcím-ellenőrzést, és a egy hasznos. A jogcím-ellenőrzés küldeni az üzenetkezelési platform, és tárolja a tartalom egy külső szolgáltatás használatával. Ez a minta lehetővé teszi, hogy a dolgozhatók fel, miközben az üzenetsínre és az ügyfél védelméről példányairól érkező és lassította le a nagy méretű üzenetek. Ez a minta is segít csökkentheti a költségeket, mivel tárolási rendszerint olcsóbb, mint az üzenetkezelési platform által használt erőforrás egységek.

Ez a minta más néven referencia-alapú üzenetkezelés, és eredetileg [leírt] [ enterprise-integration-patterns] a könyv *vállalati integrációs minták*, Gergely Hohpe és Bobby Woolf.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy üzenetkezelési-alapú architektúra valamikor küldhet, kap, és nagyméretű üzenetek kezelése képesnek kell lennie. Az ilyen üzeneteket tartalmazhat tetszőleges, beleértve a képeket is (például MRI vizsgálatok), hangfájlok (például ügyfélszolgálatával hívások), szöveges dokumentumok vagy bármilyen tetszőleges méretű bináris adatokat.

Az ilyen nagy méretű üzenetek küldése az üzenetsínre közvetlenül nem ajánlott, mert további erőforrások és a felhasznált sávszélesség van szükségük. Nagy méretű üzenetek is lelassíthatja a teljes megoldás, mivel az üzenetkezelési rendszerek általában szempontokat kisméretű üzenetet hatalmas mennyiségű kezelésére. Ezenkívül legtöbb platformokon korlátokkal rendelkeznek az üzenet mérete, így szükség lehet a nagy méretű üzenetek a korlátok megkerüléséhez.

## <a name="solution"></a>Megoldás

A teljes üzenet hasznos adattartalmából be egy külső szolgáltatás, például egy adatbázisba Store. A tárolt tartalom mutató hivatkozás beszerzése, és csak az üzenetsínre mutató küldése. A hivatkozás úgy viselkedik, mint egy jogcím ellenőrzés poggyászának, ezért egy részét lekérheti a minta neve. Az ügyfelek iránt, hogy adott üzenet feldolgozása használatával a kapott hivatkozást lekérni a tartalmat, szükség esetén.

![A jogcím-ellenőrzési mintájának ábrája](./_images/claim-check.jpg)

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Vegye figyelembe, hogy az állapotüzenet-adatokat fogyasztása, törlése, ha nincs szüksége az üzenetek archiválása. Bár a blob storage viszonylag olcsó, költségei néhány pénzt hosszú távon, különösen, ha nagy mennyiségű adatot. Az üzenet törlése végezhető szinkron módon történik az alkalmazás, amely fogadja és feldolgozza az üzenetet, vagy aszinkron módon egy külön dedikált folyamat. Az aszinkron módszer befolyásolása nélkül az átviteli sebesség és az üzenetek feldolgozási teljesítmény, a fogadó alkalmazásnak a régi adatokat távolítja el.

- Tárolását és beolvasását az üzenet hatására néhány további erőforrásokra és a késés. Érdemes a logikát alkalmazzák a a küldő alkalmazás ezt a mintát használni, csak akkor, ha az üzenet mérete meghaladja az korlát az üzenetsínre. A minta kisebb üzenetek szeretné hagyni. Ez a megközelítés a feltételes jogcím-minta eredményez.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Minden alkalommal, amikor egy üzenet nem fér el a kiválasztott üzenet bus technológia támogatott üzenet túllépi ezt a mintát kell használni. Például az Event Hubs jelenleg rendelkezik egy legfeljebb 256 KB-os (alapszintű), míg Event Grid csak 64 KB-os üzeneteket támogató.

A minta is használható, ha a tartalom legyenek elérhetők csak a látni szeretné a jogosult szolgáltatásokat. A tartalom külső erőforrásokhoz történő kiszervezésével, put körülményeket, győződjön meg arról, hogy a biztonsági kényszerítve van a tartalomban tárolt bizalmas adatok szigorúbb hitelesítési és engedélyezési szabályok.

## <a name="examples"></a>Példák

Az Azure-ban Ez a minta valósítható meg, többféle módon, és a különböző technológiákkal, de két fő kategóriába sorolhatók. Mindkét esetben a fogadó rendelkezik a feladata, hogy olvassa el a jogcím-ellenőrzés, és ezzel a hasznos adat lekéréséhez.

- **Automatikus jogcím-ellenőrzésének generációs**. Ezt a megközelítést használ [Azure Event Grid](/azure/event-grid/) automatikus létrehozásához a jogcím-ellenőrzés, és azt leküldése az üzenetsínre.

- **Manuális jogcím-ellenőrzést. generációs**. Ebben a megközelítésben a küldő feladata a tartalom kezeléséhez. A küldő a megfelelő szolgáltatás használatával a hasznos tárolja, beolvasása vagy állít elő, a jogcím-ellenőrzés, és elküldi a jogcím-ellenőrzés az üzenetsínre.

Event Grid-esemény-útválasztó szolgáltatás, és megpróbálja kézbesíti az eseményeket konfigurálható időn belüli 24 óra. Ezt követően események vagy elvetett vagy kapcsolat megszakadásának lettered. Az esemény hasznos archiválhat, vagy visszajátszani az kell, ha egy Event Grid-előfizetést is hozzáadhat az Event Hubs vagy a Queue Storage, amelyben üzeneteket hosszabb ideig kell megtartani, és üzenetek archiválása támogatott. Finom hangolási Event Grid az üzenetek kézbesítését, és próbálkozzon újra és kézbesítetlen levelek konfigurációs kapcsolatos információkért lásd: [kapcsolat megszakadásának betűs, és ismételje meg a házirendek](/azure/event-grid/manage-event-delivery).

### <a name="automatic-claim-check-generation-with-blob-storage-and-event-grid"></a>A Blob Storage és az Event Grid automatikus jogcím-ellenőrzésének generálása

Ebben a megközelítésben a küldő csökken az üzenet hasznos adattartalmából egy kijelölt Azure Blob Storage-tárolóba. Event Grid automatikusan egy címke hivatkozást hoz létre, és elküldi egy támogatott üzenetbusz, például az Azure Storage-üzenetsorokat. A fogadó lekérdezik a várólista, üzenet jelenik meg, és ezután tárolt referenciaadatokat kell felhasználnia közvetlenül a Blob Storage-ból a tartalom letöltéséhez.

Event Grid ugyanazon üzenet közvetlenül felhasználhassák [Azure Functions](/azure/azure-functions/), anélkül, hogy egy üzenetbusz keresztül. Ez a módszer teljes mértékben kihasználja az Event Grid és a Functions kiszolgáló nélküli jellegéről.

Ez a megközelítés kódpéldákat megtalálja [Itt][example-1].

### <a name="event-grid-with-event-hubs"></a>Event Grid az Event hubs szolgáltatással

Az előző példához hasonló, Event Grid automatikusan generál egy üzenetet egy hasznos adat íródik egy Azure Blob-tárolóba. Azonban ebben a példában az üzenetsínre az Event Hubs segítségével történik. Egy ügyfél saját maga által küldött üzenetek streamjét fogadásához, ahogy az eseményközpontba megírásának regisztrálhat. Az event hubs is konfigurálható úgy, hogy az üzenetek, ezzel elérhetővé téve azokat, hogy lekérdezhetők legyenek eszközökkel, mint például az Apache Spark, Apache Drill, vagy a rendelkezésre álló Avro-kódtárak használata az Avro fájllal archiválhatja.

Ez a megközelítés kódpéldákat megtalálja [Itt][example-2].

### <a name="claim-check-generation-with-service-bus"></a>A Service Bus jogcím ellenőrzése generálása

Ez a megoldás egy adott Service Bus beépülő modult, kihasználja [ServiceBus.AttachmentPlugin](https://www.nuget.org/packages/ServiceBus.AttachmentPlugin/), amely megkönnyíti a jogcím-ellenőrzésének munkafolyamat végrehajtásához. A beépülő modul, amely lekérdezi az Azure Blob Storage tárolja, az üzenet elküldésekor a melléklet bármely üzenettörzs alakítja át.

```csharp
using ServiceBus.AttachmentPlugin;
...

// Getting connection information
var serviceBusConnectionString = Environment.GetEnvironmentVariable("SERVICE_BUS_CONNECTION_STRING");
var queueName = Environment.GetEnvironmentVariable("QUEUE_NAME");
var storageConnectionString = Environment.GetEnvironmentVariable("STORAGE_CONNECTION_STRING");

// Creating config for sending message
var config = new AzureStorageAttachmentConfiguration(storageConnectionString);

// Creating and registering the sender using Service Bus Connection String and Queue Name
var sender = new MessageSender(serviceBusConnectionString, queueName);
sender.RegisterAzureStorageAttachmentPlugin(config);

// Create payload
var payload = new { data = "random data string for testing" };
var serialized = JsonConvert.SerializeObject(payload);
var payloadAsBytes = Encoding.UTF8.GetBytes(serialized);
var message = new Message(payloadAsBytes);

// Send the message
await sender.SendAsync(message);
```

A Service Bus-üzenet egy értesítési beolvasása, amely egy ügyfél előfizethetnek funkcionál. Ha a fogyasztó az üzenetet kap, a beépülő modul lehetővé teszi közvetlenül beolvashatja az adatokat Blob Storage-ból. Ezután kiválaszthatja, hogyan tovább az üzenet feldolgozására. Ez a megközelítés az egyik előnye, hogy azt kivonatolja a jogcím-ellenőrzésének munkafolyamat, a küldő és fogadó.

Ez a megközelítés kódpéldákat megtalálja [Itt][example-3].

### <a name="manual-claim-check-generation-with-kafka"></a>A Kafka manuális jogcím-ellenőrzést generálása

Ebben a példában a Kafka-ügyfelek a hasznos adatokat ír az Azure Blob Storage. Majd küld egy értesítési üzenetet a [Kakfa engedélyezve van az Event Hubs](/azure/event-hubs/event-hubs-quickstart-kafka-enabled-event-hubs). A fogyasztó fogadja az üzenetet, és a hasznos hozzáférhetnek a Blob Storage-ból. Ez a példa bemutatja, hogyan különböző üzenetküldési protokoll használható a jogcím-ellenőrzésének minta megvalósítása az Azure-ban. Szüksége lehet például meglévő Kafka-ügyfelek támogatásához.

Ez a megközelítés kódpéldákat megtalálja [Itt][example-4].

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

- A fenti példák találhatók [GitHub][sample-code].
- A vállalati integrációs minták hely rendelkezik egy [leírás] [ enterprise-integration-patterns] a mintáról.
- Egy másik példa: [foglalkoznak, nagy a Service Bus jogcím üzenetek ellenőrizze a minta](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern) (blogbejegyzés).
- Nagy méretű üzenetek kezelésére egy alternatív minta [Split] [ splitter] és [összesített][aggregator].

<!-- links -->
[aggregator]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Aggregator.html
[enterprise-integration-patterns]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html
[example-1]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-1
[example-2]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-2
[example-3]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-3
[example-4]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check/code-samples/sample-4
[sample-code]: https://github.com/mspnp/cloud-design-patterns/tree/master/claim-check
[splitter]: https://www.enterpriseintegrationpatterns.com/patterns/messaging/Sequencer.html
