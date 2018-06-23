---
title: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmushoz
description: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmus beállításához.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 77cf5d90373da2118d34301bd5c790080d3cf63f
ms.sourcegitcommit: 9a2d56ac7927f0a2bbfee07198d43d9c5cb85755
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/22/2018
ms.locfileid: "36327687"
---
# <a name="retry-guidance-for-specific-services"></a>Újrapróbálkozási útmutatás adott szolgáltatásoknál

A legtöbb Azure-szolgáltatás és ügyfél-SDK tartalmaz egy újrapróbálkozási mechanizmust. Ezek azonban eltérőek lehetnek, mert minden szolgáltatásnak eltérő jellemzői és követelményei vannak, így mindegyik újrapróbálkozási mechanizmus az adott szolgáltatásra van szabva. Ez az útmutató összefoglalást ad az Azure-szolgáltatások többségére jellemző újrapróbálkozási funkciókról, továbbá segítséget nyújt a szolgáltatás újrapróbálkozási mechanizmusának használatához, módosításához vagy kibővítéséhez.

Általános útmutatást az átmeneti hibák kezeléséhez, valamint a szolgáltatásokat és erőforrásokat célzó kapcsolatok és műveletek újrapróbálásához az [újrapróbálkozási útmutatásban](./transient-faults.md) talál.

A következő táblázat az útmutatóban érintett Azure-szolgáltatások újrapróbálkozási funkcióiról nyújt áttekintést.

| **Szolgáltatás** | **Újrapróbálkozási képességek** | **Szabályzatkonfiguráció** | **Hatókör** | **Telemetriafunkciók** |
| --- | --- | --- | --- | --- |
| **[Azure Active Directory](#azure-active-directory)** |Natív, az ADAL-kódtár része |Beágyazva az ADAL-kódtárba |Belső |None |
| **[Cosmos DB](#cosmos-db)** |Natív, a szolgáltatás része |Nem konfigurálható |Globális |TraceSource |
| **[Az Event Hubs](#azure-event-hubs)** |Natív, az ügyfél része |Szoftveres |Ügyfél |None |
| **[Redis gyorsítótár](#azure-redis-cache)** |Natív, az ügyfél része |Szoftveres |Ügyfél |TextWriter |
| **[Keresés](#azure-search)** |Natív, az ügyfél része |Szoftveres |Ügyfél |ETW vagy egyéni |
| **[Service Bus](#service-bus)** |Natív, az ügyfél része |Szoftveres |Névtérkezelő, üzenetkezelési előállító vagy ügyfél |ETW |
| **[Service Fabric](#service-fabric)** |Natív, az ügyfél része |Szoftveres |Ügyfél |None | 
| **[SQL Database with ADO.NET](#sql-database-using-adonet)** |[Polly](#transient-fault-handling-with-polly) |Deklaratív és szoftveres |Önálló utasítások vagy kódblokkok |Egyéni |
| **[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)** |Natív, az ügyfél része |Szoftveres |Alkalmazástartományonként globális |None |
| **[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)** |Natív, az ügyfél része |Szoftveres |Alkalmazástartományonként globális |None |
| **[Tárolás](#azure-storage)** |Natív, az ügyfél része |Szoftveres |Ügyfél- és különálló műveletek |TraceSource |

> [!NOTE]
> Az Azure beépített része, majd ismételje meg mechanizmusok, nincs jelenleg nem tudja alkalmazni a különböző típusú hiba vagy kivétel különböző újrapróbálkozási házirendje. Konfigurálnia kell egy házirendet, amely az optimális átlagos teljesítményt és rendelkezésre állást biztosít. A szabályzat finomhangolásához elemezze a naplófájlokat, hogy megállapítsa, milyen típusú átmeneti hibák szoktak történni. 

## <a name="azure-active-directory"></a>Azure Active Directory
Az Azure Active Directory egy átfogó, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést, a biztonsági szolgáltatásokat és az alkalmazáshozzáférés-felügyeletet. Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi szabályzatokon és szabályokon alapuló hozzáférés-vezérléssel bővíthessék az alkalmazásaikat.

> [!NOTE]
> Felügyelt Szolgáltatásidentitás végpontokon újrapróbálkozási útmutatóért lásd: [egy Azure virtuális gép felügyelt szolgáltatás Identity (MSI) használata a token beszerzési](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az Azure Active Directory beépített újrapróbálkozási mechanizmussal rendelkezik az Active Directory Authentication Library (ADAL) részeként. A váratlan zárolások elkerülése érdekében azt javasoljuk, hogy külső kódtárak és alkalmazáskódok **ne** próbálkozhassanak újra sikertelen kapcsolódás esetén, és ezek újrapróbálkozásait az ADAL kezelje. 

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre az Azure Active Directory használata során:

* Amikor csak lehetséges, az ADAL-kódtárat és az újrapróbálkozások beépített támogatását használja.
* Ha a REST API-t használ az Azure Active Directory, a eredménykódja 429 (túl sok kérelem) vagy a 5xx tartomány hiba esetén próbálja megismételni a műveletet. Más hibák esetében ne engedélyezze az újrapróbálkozást.
* Az exponenciális visszatartási szabályzat használatát az Azure Active Directory Batch-forgatókönyvei esetében javasoljuk.

A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez. Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.

| **Környezet** | **Mintacél E2E<br />max. késleltetése** | **Újrapróbálkozási stratégia** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felület<br />vagy előtér |2 másodperc |FixedInterval |Ismétlések száma<br />Újrapróbálkozási időköz<br />Első gyors újrapróbálkozás |3<br />500 ms<br />true |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – 500 ms késleltetés<br />3. kísérlet – 500 ms késleltetés |
| Háttér vagy<br />kötegelt |60 másodperc |ExponentialBackoff |Ismétlések száma<br />Visszatartás (min.)<br />Visszatartás (max.)<br />Visszatartás (változás)<br />Első gyors újrapróbálkozás |5<br />0 másodperc<br />60 másodperc<br />2 másodperc<br />false |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – kb. 2 mp. késleltetés<br />3. kísérlet – kb. 6 mp. késleltetés<br />4. kísérlet – kb. 14 mp. késleltetés<br />5. kísérlet – kb. 30 mp. késleltetés |

### <a name="more-information"></a>További információ
* [Az Azure Active Directory hitelesítési kódtárai][adal]

## <a name="cosmos-db"></a>Cosmos DB

A Cosmos DB egy teljes körűen felügyelt, többmodelles adatbázis-szolgáltatás, amely támogatja a séma nélküli JSON-adatok használatát. Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletekkel. Az újrapróbálkozások számának és a maximális várakozási idő beállításához a [ConnectionPolicy.RetryOptions] konfigurálása szükséges. Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.

Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza. Ellenőrizze a `DocumentClientException` állapotkódját.

### <a name="policy-configuration"></a>Szabályzatkonfiguráció
A következő táblázatban a `RetryOptions` osztály alapértelmezett beállításait tekintheti meg.

| Beállítás | Alapértelmezett érték | Leírás |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Az újrapróbálkozások maximális száma abban az esetben, ha a Cosmos DB az ügyfélre alkalmazott korlátozása miatt sikertelen a kísérlet. |
| MaxRetryWaitTimeInSeconds |30 |A maximális újrapróbálkozási idő másodpercben. |

### <a name="example"></a>Példa
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Telemetria
Az újrapróbálkozási kísérletek strukturálatlan nyomkövetési üzenetekként lesznek naplózva a .NET **TraceSource** használatával. Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.

Amennyiben például a következőt adja hozzá az App.config fájlhoz, a szövegfájlban a végrehajtható fájllal megegyező helyen jönnek létre a nyomkövetési adatok:

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a>Event Hubs

Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az Azure Event Hubs Client Library újrapróbálkozási viselkedését az `EventHubClient` osztály `RetryPolicy` tulajdonsága vezérli. Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Event Hub egy átmeneti `EventHubsException` vagy egy `OperationCanceledException` választ ad.

### <a name="example"></a>Példa
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>További információ
[.NET standard ügyféloldali kódtár az Azure Event Hubshoz](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a>Azure Redis Cache
Az Azure Redis Cache gyors adathozzáférést és alacsony késést kínáló gyorsítótár-szolgáltatás, amely a népszerű, nyílt forráskódú Redis Cache-re épül. Biztonságos, a Microsoft felügyeli, és az Azure bármelyik alkalmazásából elérhető.

Ebben az útmutatóban azt feltételezzük, hogy a StackExchange.Redis ügyfelet használja a gyorsítótár eléréséhez. A további alkalmas ügyfelek listája a [Redis webhelyén](http://redis.io/clients) tekinthető meg, ám ezeknek eltérő újrapróbálkozási mechanizmusai lehetnek.

Vegye figyelembe, hogy a StackExchange.Redis ügyfél egyetlen kapcsolaton keresztül végez multiplexálást. A javasolt felhasználás az, ha létrehozza az ügyfél egy példányát az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár elérését célzó összes művelethez. Így a gyorsítótárral való kapcsolat csak egyszer jön létre, ezért az ebben a szakaszban leírt összes útmutatás ezen első kapcsolat újrapróbálkozási szabályzatára vonatkozik, nem pedig a gyorsítótárhoz hozzáférő egyes műveletekre.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
A StackExchange.Redis ügyfél egy számos beállítással konfigurált kapcsolatkezelő osztályt használ. Néhány példa ezekre a beállításokra:

- **ConnectRetry**. Ennyi alkalommal próbálkozik újra a gyorsítótárhoz való sikertelen kapcsolódás esetén.
- **ReconnectRetryPolicy**. A használt újrapróbálkozási stratégia.
- **ConnectTimeout**. A maximális várakozási idő milliszekundumban.

### <a name="policy-configuration"></a>Szabályzatkonfiguráció
Az újrapróbálkozási szabályzat konfigurálása szoftveresen történik. Az ügyfél beállításait a gyorsítótárhoz való kapcsolódás előtt kell megadni. Ehhez létre kell hozni a **ConfigurationOptions** osztály egy példányát, feltölteni adatokkal a tulajdonságait, majd továbbítani azt a **Connect** metódusnak.

A beépített osztályok támogatják a lineáris (állandó) késleltetést, illetve az exponenciális visszatartást véletlenszerű újrapróbálkozási időközökkel. Ezenkívül létrehozhat egyéni újrapróbálkozási szabályzatot az **IReconnectRetryPolicy** felület implementálásával.

A következő példa exponenciális visszatartással konfigurálja az újrapróbálkozási stratégiát.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Egy másik lehetőség, hogy a beállításokat sztringként adja meg és továbbítja a **Connect** metódusnak. Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezen a módon, csak a programkódon keresztül.

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Amikor a gyorsítótárhoz csatlakozik, közvetlenül is megadhat beállításokat.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

További információért tekintse meg [a Stack Exchange Redis konfigurálását](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.

A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.

| **Környezet** | **Beállítás** | **Alapértelmezett érték**<br />(v 1.2.2) | **Jelentés** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Legfeljebb 5000 ms, plusz SyncTimeout<br />1000<br /><br />LinearRetry 5000 ms |Ennyi alkalommal kell megismételni a csatlakozási kísérletet az első csatlakozási művelet során.<br />A csatlakozási műveletek időtúllépési értéke (ms). Nem az újrapróbálkozás kísérletek közötti késleltetést jelzi.<br />A szinkron műveletek számára biztosított idő (ms).<br /><br />Újrapróbálkozás 5000 ms időközönként.|

> [!NOTE]
> A szinkron műveletek esetében a `SyncTimeout` hozzájárulhat a végpontok közötti késéshez, de a túl alacsony érték gyakori időtúllépéseket eredményezhet. További információért lásd [az Azure Redis Cache hibaelhárítását][redis-cache-troubleshoot]. Általában kerülje a szinkron műveletek használatát, és alkalmazzon inkább aszinkron műveleteket. További információért lásd a [folyamatokat és a multiplexereket](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).
>
>

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre az Azure Redis Cache használata során:

* A StackExchange Redis ügyfél kezeli a saját újrapróbálkozásait, de csak amikor az alkalmazás első indításakor próbál kapcsolódni a gyorsítótárhoz. Meghatározhatja a kapcsolati időtúllépés értékét, az újrapróbálkozási kísérletek számát, valamint a kapcsolat létrehozására tett ismételt próbálkozások között eltelt időt, de az újrapróbálkozási szabályzat nem vonatkozik a gyorsítótárra irányuló műveletekre.
* Nagy számú újrapróbálkozási kísérlet helyett érdemes lehet inkább az eredeti adatforráshoz csatlakozni.

### <a name="telemetry"></a>Telemetria
**TextWriter** használatával adatokat gyűjthet a kapcsolatokról (más műveletekről azonban nem).

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Az alábbi példa azt mutatja be, hogy ez milyen kimenetet eredményez.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Példák
A következő mintakód állandó (lineáris) késleltetést állít be az újrapróbálkozások között a StackExchange.Redis ügyfél inicializálásakor. Ez a példa azt mutatja be, hogyan állítható be a konfiguráció egy **ConfigurationOptions**-példánnyal.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

A következő példa sztringként adja meg a beállításokat, és így határozza meg a konfigurációt. A kapcsolat időtúllépése a leghosszabb idő, amennyit a kapcsolat a gyorsítótárra várhat, nem pedig az újrapróbálkozási kísérletek közötti időköz. Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonságot csak a programkódon keresztül lehet beállítani.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

További példákért tekintse meg a projekt webhelyének a [konfigurációval](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) foglalkozó szakaszát.

### <a name="more-information"></a>További információ
* [Redis-webhely](http://redis.io/)

## <a name="azure-search"></a>Azure Search
Az Azure Search hatékony és kifinomult keresési lehetőségekkel egészíthet ki egy webhelyet vagy alkalmazást, gyorsan és könnyen pontosítja a keresési eredményeket, továbbá részletes és finomhangolt rangsorolási modelleket képes létrehozni.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az Azure Search SDK újrapróbálkozási viselkedését a [SearchServiceClient] és a [SearchIndexClient] osztály `SetRetryPolicy` metódusa vezérli. Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Search 5xx-es vagy 408-as (Kérés időtúllépése) választ ad vissza.

### <a name="telemetry"></a>Telemetria
Nyomkövetés ETW-vel vagy egyéni nyomkövetési szolgáltató regisztrálásával. További információt az [AutoRest dokumentációjában][autorest] találhat.

## <a name="service-bus"></a>Service Bus
A Service Bus egy felhőalapú üzenetkezelési platform, amely skálázható és rugalmas módon biztosít lazán kapcsolódó üzenetváltásokat az alkalmazások összetevői számára, legyen szó felhőalapú vagy helyszíni megoldásról.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
A Service Bus a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály implementációi alapján implementálja az újrapróbálkozásokat. Az összes Service Bus-ügyfél elérhetővé tesz egy **RetryPolicy** tulajdonságot, amely beállítható a **RetryPolicy** alaposztály egyik implementációjaként. A beépített implementációk a következők:

* A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). Ez elérhetővé teszi azokat a tulajdonságokat, amelyek a visszatartási időközöket, az újrapróbálkozások számát és a **TerminationTimeBuffer** tulajdonságot szabályozzák, amely korlátozza, hogy a művelet hányszor hajtható végre.
* A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). Ezt akkor szokták használni, ha nincs szükség újrapróbálkozásra a Service Bus API-szintjén, például ha az újrapróbálkozásokat egy másik folyamat kezeli egy kötegelt vagy többlépéses művelet részeként.

A Service Bus műveletek visszatérhet a kivételek, számos, a [Service Bus üzenetküldési kivételek](/azure/service-bus-messaging/service-bus-messaging-exceptions). A lista azt is ismerteti, hogy ezek közül melyik utal arra, hogy a művelet újrapróbálható. Például a **ServerBusyException** azt jelzi, hogy az ügyfélnek várnia kell egy ideig, majd ismét megpróbálkozni a művelettel. A **ServerBusyException** jelentkezésekor a Service Bus eltérő módba vált, amelyben további 10 másodperc adódik a kiszámított újrapróbálkozási késleltetéshez. Ebből a módból rövid időn belül automatikusan kilép.

A Service Bus által visszaadott kivételek elérhetővé teszik az **IsTransient** tulajdonságot, amely jelzi, hogy az ügyfélnek érdemes-e újrapróbálkoznia a művelettel. A beépített **RetryExponential** szabályzat a **MessagingException** osztály (az összes Service Bus-kivétel alaposztálya) **IsTransient** tulajdonságára hagyatkozik. A **RetryPolicy** alaposztály egyéni implementációi esetében a kivételtípus és az **IsTransient** tulajdonság közös használatával pontosabban szabályozhatja az újrapróbálkozási műveleteket. Például észlelhet egy **QuotaExceededException**-kivételt, és utasítást adhat, hogy csak az üzenetsor kiürítése után próbálkozzon újra az üzenetküldéssel.

### <a name="policy-configuration"></a>Szabályzatkonfiguráció
Az újrapróbálkozási szabályzatok beállítása szoftveresen történik, és beállítható alapértelmezett szabályzatként a **NamespaceManager** és a **MessagingFactory** számára, vagy egyenként, az egyes üzenetkezelési ügyfelek számára. Az üzenetkezelési munkamenet alapértelmezett újrapróbálkozási szabályzatát a **NamespaceManager** **RetryPolicy** tulajdonságában állíthatja be.

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

Az üzenetkezelési előállítóból létrehozott ügyelek alapértelmezett újrapróbálkozási szabályzatát a **MessagingFactory** **RetryPolicy** tulajdonságában állíthatja be.

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

Az üzenetkezelési ügyfél alapértelmezett újrapróbálkozási szabályzatának beállításához, illetve az alapértelmezett szabályzat felülbírálásához a szükséges szabályzatosztály példányának **RetryPolicy** tulajdonságát kell megadnia:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

Az újrapróbálkozási szabályzat nem állítható be az egyes műveletek szintjén. Az az üzenetkezelési ügyfél összes műveletére érvényes lesz.
A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.

| Beállítás | Alapértelmezett érték | Jelentés |
|---------|---------------|---------|
| Szabályzat | Exponenciális | Exponenciális visszatartás. |
| MinimalBackoff | 0 | Minimális visszatartási időköz. Ez hozzáadódik a deltaBackoff alapján kiszámított újrapróbálkozási időközhöz. |
| MaximumBackoff | 30 másodperc | Maximális visszatartási időköz. A MaximumBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke. |
| DeltaBackoff | 3 másodperc | Az újrapróbálkozások közötti visszatartási időköz. Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét fogja használni a rendszer. |
| TimeBuffer | 5 másodperc | Az újrapróbálkozáshoz tartozó leállítási időpuffer. A rendszer felhagy az újrapróbálkozással, ha a TimeBuffer értékben megadottnál kevesebb idő van hátra. |
| MaxRetryCount | 10 | Az újrapróbálkozások maximális száma. |
| ServerBusyBaseSleepTime | 10 másodperc | Ha az utolsó kivétel a **ServerBusyException** volt, ez az érték hozzáadódik a kiszámított újrapróbálkozási időközhöz. Ez az érték nem módosítható. |

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre a Service Bus használata során:

* A beépített **RetryExponential** implementáció használatakor nincs szükség tartalékműveletek megvalósítására, mivel a szabályzat a „foglalt kiszolgáló” kivételekre reagálva automatikusan átvált a megfelelő újrapróbálkozási módra.
* A Service Bus támogatja a Párosított névterek nevű funkciót, amely automatikus feladatátvételt implementál, és az elsődleges névtér üzenetsorának hibájakor egy másik névtér tartalék üzenetsorára vált. A másodlagos üzenetsor üzenetei továbbküldhetők az elsődleges üzenetsornak, miután az helyreállt. Ez a funkció az átmeneti hibák kezelésére szolgál. További információért lásd [az aszinkron üzenetkezelési minták és a magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx) ismertetését.

A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez. Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.

| Környezet | Példa a maximális késésre | Újrapróbálkozási szabályzat | Beállítások | Működés |
|---------|---------|---------|---------|---------|
| Interaktív, felhasználói felület vagy előtér | 2 másodperc*  | Exponenciális | MinimumBackoff = 0 <br/> MaximumBackoff = 30 mp. <br/> DeltaBackoff = 300 ms <br/> TimeBuffer = 300 ms <br/> MaxRetryCount = 2 | 1. kísérlet: 0 mp. késleltetés <br/> 2. kísérlet: kb. 300 ms késleltetés <br/> 3. kísérlet: kb. 900 ms késleltetés |
| Háttér vagy kötegelt | 30 másodperc | Exponenciális | MinimumBackoff = 1 <br/> MaximumBackoff = 30 mp. <br/> DeltaBackoff = 1,75 mp. <br/> TimeBuffer = 5 mp. <br/> MaxRetryCount = 3 | 1. kísérlet: kb. 1 mp. késleltetés <br/> 2. kísérlet: kb. 3 mp. késleltetés <br/> 3. kísérlet: kb. 6 ms késleltetés <br/> 4. kísérlet: kb. 13 ms késleltetés |

\* Nem tartalmazza a további késleltetést, amely a „foglalt kiszolgáló” válasz esetén adódik az értékhez.

### <a name="telemetry"></a>Telemetria
A Service Bus ETW-eseményként naplózza az újrapróbálkozásokat egy **EventSource** használatával. Egy **EventListener** az eseményforráshoz csatolása szükséges, ha rögzíteni kívánja az eseményeket, és meg kívánja tekinteni azokat a Teljesítménynaplóban, vagy ha egy megfelelő célnaplóba írná azokat. Ehhez a [szemantikus naplózási alkalmazásblokk](http://msdn.microsoft.com/library/dn775006.aspx) használatát javasoljuk. Az újrapróbálkozási események formátuma a következő:

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Példák
A következő mintakód bemutatja, hogyan állíthatja be az újrapróbálkozási szabályzatot a következőkhöz:

* Egy névtérkezelő. A szabályzat a kezelő összes műveletére vonatkozik, és nem bírálható felül az egyes műveletek esetében.
* Egy üzenetkezelési előállító. A szabályzat az előállítóból létrehozott összes ügyfélre vonatkozik, és nem bírálható felül az egyes ügyfelek létrehozásakor.
* Egy különálló üzenetkezelési ügyfél. Az ügyfél létrehozása után beállíthatja annak újrapróbálkozási szabályzatát. A szabályzat az ügyfél összes műveletére vonatkozik.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>További információ
* [Aszinkron üzenetkezelési minták és magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a>Service Fabric

A megbízható szolgáltatások Service Fabric-fürtön belüli elosztásával a cikkben tárgyalt legtöbb átmeneti hiba elkerülhető. Azonban így is előfordulhatnak átmeneti hibák. Előfordulhat például, hogy a kérés érkezésekor az elnevezési szolgáltatás egy útválasztási változtatást végez, ami kivételt eredményez. Ugyanez a kérés 100 milliszekundummal később talán sikeres lett volna.

A Service Fabric az ehhez hasonló átmeneti hibák belső kezelését elvégzi. A szolgáltatás beállítása közben konfigurálhat egyes beállításokat az `OperationRetrySettings` osztállyal.  Az alábbi kód erre mutat egy példát. A legtöbb esetben ez nem szükséges, és az alapértelmezett beállítások tökéletesen megfelelnek.

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a>További információ

* [Távoli kivételkezelés](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a>Az ADO.NET használatával SQL-adatbázis
Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az SQL Database nem tartalmaz beépített támogatást az újrapróbálkozásokhoz, ha az ADO.NET használatával érik el. Ugyanakkor a kérések válaszkódjából megállapítható, hogy a kérés miért hiúsult meg. További információt az SQL Database szabályozásáról [az Azure SQL Database erőforrás-korlátait](/azure/sql-database/sql-database-resource-limits) ismertető szakaszban talál. A kapcsolódó hibakódok listáját [az SQL Database ügyfélalkalmazásaiban felmerülő SQL-hibakódokat](/azure/sql-database/sql-database-develop-error-messages) ismertető szakaszban találja.

A Polly kódtárt alkalmazva implementálhatja az újrapróbálkozást az SQL Database-ben. További információt az [átmeneti hibák a Polly használatával történő kezelését](#transient-fault-handling-with-polly) ismertető szakaszban talál.

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre, amikor az ADO.NET használatával éri el az SQL Database-t:

* Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium). A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják. Ha kiszámíthatóbb teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.
* Gondoskodjon arról, hogy az újrapróbálkozás a megfelelő szinten vagy hatókörrel legyen végrehajtva, amivel elkerülheti, hogy a nem idempotens műveletek miatt inkonzisztencia keletkezzen az adatokban. Ideális esetben minden műveletnek idempotensnek kellene lennie, hogy inkonzisztencia veszélye nélkül lehessen ismételni azokat. Ellenkező esetben az újrapróbálkozást olyan szinten vagy hatókörrel kell végrehajtani, hogy az összes kapcsolódó változtatást vissza lehessen vonni egy művelet meghiúsulásakor – például egy tranzakciós hatókörben. További információt a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee) foglalkozó részében talál.
* Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását. Az interaktív forgatókönyvek kivételt képeznek, mivel csak néhány újrapróbálkozás történik nagyon rövid időközökkel. Ehelyett a legtöbb esetben exponenciális visszatartási stratégia használata javasolt.
* A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez. A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt. A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást. Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Gyakorlatilag hozzáadódik az újrapróbálkozási szabályzatban megadott újrapróbálkozási késleltetéshez minden egyes újrapróbálkozási kísérlet esetén.
* Megszakítja a kapcsolatot adott számú újrapróbálkozás után, akár egy exponenciális visszatartási újrapróbálkozási logika használata esetén is, és egy új kapcsolatot létesítve próbálja meg újra végrehajtani a műveletet. Ha ugyanazzal a művelettel többször próbálkozik újra ugyanazon a kapcsolaton keresztül, az önmagában is csatlakozási problémákat okozhat. Erre a technikára egy példát a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx) foglalkozó részében találhat.
* Ha kapcsolatkészletezést alkalmaz (ez az alapértelmezett beállítás), akkor előfordulhat, hogy a készletből ismét ugyanazt a kapcsolatot választja a rendszer, akár a kapcsolat lezárása és ismételt megnyitása után is. Ebben az esetben az **SqlConnection** osztályból kell meghívni a **ClearPool** metódust, és jelezni, hogy a kapcsolat nem újrahasználható. Ezt azonban csak abban az esetben javasoljuk, ha számos csatlakozási kísérlet meghiúsult, és csak akkor, ha az átmeneti hibák, például SQL-időtúllépések (-2-es hibakód) adott osztálya hibás kapcsolatokhoz kötődik.
* Amennyiben az adatelérési kód **TransactionScope**-példányként kezdeményezett tranzakciókat alkalmaz, az újrapróbálkozási logikának újra meg kell nyitnia a kapcsolatot, és új tranzakció-hatókört kell kezdeményeznie. Ezért az újrapróbálható kódblokknak a tranzakció teljes hatókörét le kell fedne.

A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez. Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.

| **Környezet** | **Mintacél E2E<br />max. késleltetése** | **Újrapróbálkozási stratégia** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felület<br />vagy előtér |2 másodperc |FixedInterval |Ismétlések száma<br />Újrapróbálkozási időköz<br />Első gyors újrapróbálkozás |3<br />500 ms<br />true |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – 500 ms késleltetés<br />3. kísérlet – 500 ms késleltetés |
| Háttér<br />vagy kötegelt |30 másodperc |ExponentialBackoff |Ismétlések száma<br />Visszatartás (min.)<br />Visszatartás (max.)<br />Visszatartás (változás)<br />Első gyors újrapróbálkozás |5<br />0 másodperc<br />60 másodperc<br />2 másodperc<br />false |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – kb. 2 mp. késleltetés<br />3. kísérlet – kb. 6 mp. késleltetés<br />4. kísérlet – kb. 14 mp. késleltetés<br />5. kísérlet – kb. 30 mp. késleltetés |

> [!NOTE]
> A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz. Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.
>
>

### <a name="examples"></a>Példák
Ebben a szakaszban azt mutatjuk be, hogy miként használhatja a Pollyt az Azure SQL Database elérésére a `Policy` osztályban konfigurált újrapróbálkozási szabályzatok segítségével.

A következő programkód bemutatja, hogyan bővítheti az `SqlCommand` osztályt, ha az exponenciális visszatartással hívja meg az `ExecuteAsync` metódust.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

Ezt az aszinkron bővítési metódust a következőképpen lehet használni.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>További információ
* [Cloud Service Fundamentals adatelérési réteg – átmeneti hibák kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Ha további útmutatásra van szüksége az SQL Database minél jobb kihasználásához, olvassa el [az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatóját](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).

## <a name="sql-database-using-entity-framework-6"></a>SQL-adatbázis használata az Entity Framework 6
Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető. Az Entity Framework egy objektumrelációs leképező .NET-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet relációs adatokkal dolgozni. Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el, a [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmus segítségével. Az újrapróbálkozási mechanizmus fő funkciói a következők:

* Az elsődleges absztrakció az **IDbExecutionStrategy** felület. Ez a felület:
  * Definiálja a szinkron és aszinkron **feldolgozási*** metódusokat.
  * Definiálja azokat az osztályokat, amelyek felhasználhatók közvetlenül, illetve konfigurálhatók alapértelmezett stratégiaként egy adatbázis-környezetben, leképezhetők egy szolgáltatónévre, vagy pedig leképezhetők egy szolgáltatónévre vagy kiszolgálónévre. Ha egy környezethez konfigurálják, az újrapróbálkozások az egyes adatbázis-műveletek szintjén történnek, amelyekből több is lehet egy adott környezeti művelet esetében.
  * Definiálja, hogy a sikertelen csatlakozást mikor kövesse újrapróbálkozás, és hogyan.
* Az **IDbExecutionStrategy** felület számos beépített implementálását tartalmazza:
  * Alapértelmezett – nincs újrapróbálkozás.
  * Az SQL Database alapértelmezett beállítása (automatikus) – nincs újrapróbálkozás, de megvizsgálja a kivételeket, és beburkolja azokat, az SQL Database stratégia használatát javasolva.
  * Az SQL Database alapértelmezett beállítása – exponenciális (az alaposztálytól örökölt), plusz az SQL Database észlelési logikája.
* Véletlenszerűsítést tartalmazó exponenciális visszatartási stratégiát implementál.
* A beépített újrapróbálkozási osztályok állapotalapúak, és nem alkalmasak a többszálú futtatásra. Ugyanakkor az aktuális művelet befejezése után újra felhasználhatók.
* Amennyiben az újrapróbálkozások száma meghaladja a megadott értéket, a szolgáltatás új kivételbe burkolja az eredményeket. Nem rendezi buborékba az aktuális kivételt.

### <a name="policy-configuration"></a>Szabályzatkonfiguráció
Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el. Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik. A konfiguráció nem módosítható az egyes műveletek szintjén.

Ha egy környezetfüggő stratégiát tesz alapértelmezetté, megad egy függvényt, amely igény szerint hoz létre egy új stratégiát. A következő kód azt mutatja be, hogyan hozható létre egy újrapróbálkozási konfigurációs osztályt, amely kibővíti a **DbConfiguration** alaposztályt.

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

Az alkalmazás indításakor, a **DbConfiguration**-példány **SetConfiguration** metódusával teheti az alapértelmezett újrapróbálkozási stratégiává minden művelet számára. Alapértelmezés szerint az EF automatikusan észleli és használatba veszi a konfigurációs osztályt.

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

Ha meg kíván adni egy újrapróbálkozási konfigurációs osztályt a környezethez, lássa el a környezeti osztályt egy **DbConfigurationType** attribútummal. Amennyiben csak egy konfigurációs osztálya van, az EF azt fogja használni, nem szükséges külön jeleznie ezt.

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

Ha eltérő újrapróbálkozási stratégiára van szüksége adott műveletek esetében, vagy le kívánja tiltani az újrapróbálkozásokat egyes műveleteknél, létrehozhat egy konfigurációs osztályt, amely lehetővé teszi stratégiák felfüggesztését vagy cseréjét egy jelző a **CallContext** osztályban történő elhelyezésével. A konfigurációs osztály ezt a jelzőt használva vált stratégiát vagy tiltja le a felhasználó által megadott stratégiát, majd az alapértelmezett stratégiát használja. További információt a végrehajtási stratégiák újrapróbálkozásainak korlátozásait ismertető oldal [végrehajtási stratégia felfüggesztését](http://msdn.microsoft.com/dn307226#transactions_workarounds) ismertető szakaszában talál (EF6-os vagy újabb verzió).

Bizonyos műveletek esetében egy másik lehetőség adott újrapróbálkozási stratégia használatára, ha létrehozza a kívánt stratégiaosztály egy példányát, és paramétereket használva ellátja a kívánt beállításokkal. Ezután meghívja annak **ExecuteAsync** metódusát.

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

A **DbConfiguration** osztály használatának legegyszerűbb módja, ha megkeresi ugyanazt a szerelvényt, amely a **DbContext** osztályban szerepel. Ez azonban nem alkalmazható, ha különböző forgatókönyvekben van szükség ugyanarra a környezetre, például a különböző interaktív és háttérbeli újrapróbálkozási stratégiák esetében. Ha a különböző környezetek különálló alkalmazástartományokban futnak, igénybe veheti a konfigurációs osztályok a konfigurációs fájlban történő megadásának beépített lehetőségét, illetve beállíthatja azt közvetlenül is a kódban. Ha a különböző környezeteknek ugyanabban az alkalmazástartományban kell futniuk, egyéni megoldásra lesz szükség.

További információt a [kódalapú konfigurálást (EF6-os vagy újabb verzió)](http://msdn.microsoft.com/data/jj680699.aspx) ismertető szakaszban találhat.

A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók az EF6 használatakor.

| Beállítás | Alapértelmezett érték | Jelentés |
|---------|---------------|---------|
| Szabályzat | Exponenciális | Exponenciális visszatartás. |
| MaxRetryCount | 5 | Az újrapróbálkozások maximális száma. |
| MaxDelay | 30 másodperc | Az újrapróbálkozások közötti maximális késleltetés. Az érték nem befolyásolja az egymást követő késleltetések kiszámítását. Csak a felső határt adja meg. |
| DefaultCoefficient | 1 másodperc | Az exponenciális visszatartás számításának együtthatója. Ez az érték nem módosítható. |
| DefaultRandomFactor | 1.1 | A véletlenszerű késleltetés bejegyzésekhez való hozzáadására szolgáló szorzó. Ez az érték nem módosítható. |
| DefaultExponentialBase | 2 | A következő késleltetés kiszámítására szolgáló szorzó. Ez az érték nem módosítható. |

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre, amikor az EF6 használatával éri el az SQL Database-t:

* Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium). A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják. Ha kiszámítható teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.
* Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását. Használjon inkább egy exponenciális visszatartási stratégiát, mivel a szolgáltatás túlterhelődhet, és a hosszabb késleltetés több időt ad a helyreállításra.
* A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez. Az időtúllépés ideális értékét saját üzleti logikájának felépítése és tesztek alapján állapíthatja meg. A későbbiekben szükség lehet az érték módosítására, ahogy változik az adatmennyiség, vagy változnak az üzleti folyamatok. A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt. A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást. Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Bár nehéz előre megállapítani, hogy hány parancsot kell végrehajtani a környezet mentésekor. Az alapértelmezett időtúllépést a **DbContext** példány **CommandTimeout** tulajdonságában adhatja meg.
* Az Entity Framework támogatja a konfigurációs fájlokban definiált újrapróbálkozási konfigurációk használatát. Ugyanakkor a minél nagyobb rugalmasság érdekében azt javasoljuk, hogy a konfigurációt szoftveresen hozza létre az alkalmazásban. Az újrapróbálkozási szabályzatok konkrét paraméterei – például az újrapróbálkozások száma és az újrapróbálkozási időközök – a szolgáltatás konfigurációs fájljában tárolhatók, és a futtatás során felhasználhatók a megfelelő szabályzatok létrehozására. Így a beállítások az alkalmazás újraindítása nélkül módosíthatók.

A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez. Az újrapróbálkozási kísérletek közötti késleltetés nem adható meg (rögzített, és egy exponenciális sorozat részeként jön létre). Ha nem hoz létre egyéni újrapróbálkozási stratégiát, csak a maximális értékeket adhatja meg, az itt látható módon. Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.

| **Környezet** | **Mintacél E2E<br />max. késleltetése** | **Újrapróbálkozási szabályzat** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felület<br />vagy előtér |2 másodperc |Exponenciális |MaxRetryCount<br />MaxDelay |3<br />750 ms |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – 750 ms késleltetés<br />3. kísérlet – 750 ms késleltetés |
| Háttér<br /> vagy kötegelt |30 másodperc |Exponenciális |MaxRetryCount<br />MaxDelay |5<br />12 másodperc |1. kísérlet – 0 mp. késleltetés<br />2. kísérlet – kb. 1 mp. késleltetés<br />3. kísérlet – kb. 3 mp. késleltetés<br />4. kísérlet – kb. 7 mp. késleltetés<br />5. kísérlet – 12 mp. késleltetés |

> [!NOTE]
> A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz. Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.
>
>

### <a name="examples"></a>Példák
A következő mintakód egy egyszerű adatelérési megoldást definiál, amely az Entity Frameworköt használja. Egy adott újrapróbálkozási stratégiát állít be a **BlogConfiguration** osztály egy példányának definiálásával, amely a **DbConfiguration** osztályt bővíti ki.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

A [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmussal foglalkozó témakörben további példákat talál az Entity Framework újrapróbálkozási mechanizmusának használatára.

### <a name="more-information"></a>További információ
* [Az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatója](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a>SQL-adatbázis használata az Entity Framework Core
Az [Entity Framework Core](/ef/core/) egy objektumrelációs leképező .NET Core-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet adatokkal dolgozni. Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk. Az Entity Framework e verzióját az alapoktól építettük újra, ezért nem örökli meg automatikusan az EF6.x összes funkcióját.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework Core-ral érik el, a [kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency) mechanizmus segítségével. A kapcsolat rugalmassága először az EF Core 1.1.0-ban vált elérhetővé.

Az elsődleges absztrakció az `IExecutionStrategy` felület. Az SQL Server, és így az SQL Azure végrehajtási stratégiája felismeri a kivételtípusokat, amelyeknél elérhető az újrapróbálkozás, és észszerű alapértelmezett értékek vannak beállítva többek között az újrapróbálkozások maximális számánál és az újrapróbálkozások közötti késleltetésnél.

### <a name="examples"></a>Példák

A következő kód lehetővé teszi az automatikus újrapróbálkozást a DbContext objektum konfigurálásakor, ami egy munkamenetet jelöl az adatbázisban. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

A következő kód bemutatja, egy végrehajtási stratégia alkalmazásával hogyan hajtható végre egy tranzakció automatikus újrapróbálkozással. A tranzakciót egy delegált definiálja. Átmeneti hiba előfordulásakor a végrehajtási stratégia ismét meghívja a delegáltat.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a>Azure Storage
Az Azure Storage szolgáltatásai közé tartoznak a tábla- és blobtárolók, a fájl- és tárolási üzenetsorok.

### <a name="retry-mechanism"></a>Újrapróbálkozási mechanizmus
Újrapróbálkozásokra az egyes REST-műveletek szintjén kerül sor, és az ügyfél-API implementálásának szerves részeit képezik. Az ügyfél Storage SDK-ja az [IExtendedRetryPolicy felületet](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx) implementáló osztályokat alkalmaz.

A felület több módon implementálható. A Storage-ügyfelek kifejezetten táblákra, blobokra és üzenetsorokra szabott szabályzatok közül választhatnak. Minden implementáció különböző újrapróbálkozási stratégiát használ, amely alapvetően az újrapróbálkozási időközöket és egyéb részleteket határoz meg.

A beépített osztályok támogatják a lineáris (állandó késleltetés) és az exponenciális (véletlenszerű újrapróbálkozási időközök) konfigurációkat. Nincs szükség újrapróbálkozási szabályzat beállítására, ha egy másik folyamat magasabb szinten kezeli az újrapróbálkozásokat. Ugyanakkor implementálhat saját újrapróbálkozási osztályokat, ha a beépített osztályok nem felelnek meg valamilyen konkrét követelménynek.

A váltakozó újrapróbálkozás a tárolási szolgáltatás elsődleges és másodlagos helye között vált, ha írásvédett georedundáns tárolót (RA-GRS) használ, és a kérés eredménye egy újrapróbálható hiba. További információt [az Azure Storage redundanciabeállításainál](http://msdn.microsoft.com/library/azure/dn727290.aspx) találhat.

### <a name="policy-configuration"></a>Szabályzatkonfiguráció
Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik. Megszokott eljárás egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** vagy **QueueRequestOptions** példány létrehozása és adatokkal történő feltöltése.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

Beállíthat egy kérésbeállítási példányt az ügyfélen, és ezt követően az ügyfelet érintő műveletek az így megadott kérésbeállításokat fogják használni.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Az ügyfél kérésbeállításainak felülbírálásához a kérésbeállítás-osztály egy adatokkal feltöltött példányát kell megadni paraméterként a műveleti metódusoknál.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Az **OperationContext** példányban határozhatja meg, hogy milyen kódot kell végrehajtani újrapróbálkozás esetén, illetve ha befejeződik a művelet. Ez a kód gyűjti össze a művelettel kapcsolatos adatokat, amelyek bekerülnek a naplókba és a telemetriába.

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

A kiterjesztett újrapróbálkozási szabályzatok jelzik, hogy egy adott hiba után szükség van-e újrapróbálkozásra, ezenkívül egy **RetryContext** objektumot adnak vissza, amely megmutatja az újrapróbálkozások számát, a legutóbbi kérés eredményét, illetve hogy a következő újrapróbálkozás az elsődleges vagy a másodlagos helyen fog történni (a részleteket lásd az alábbi táblázatban). A **RetryContext** objektummal lehet meghatározni, hogy történjen-e újrapróbálkozás, és ha igen, mikor. További információért lásd az [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx) ismertetését.

A következő táblázatban a beépített újrapróbálkozási szabályzatok alapértelmezett beállításai tekinthetők meg.

**Kérésbeállítások**

| **Beállítás** | **Alapértelmezett érték** | **Jelentés** |
| --- | --- | --- |
| MaximumExecutionTime | None | A kérés maximális végrehajtási ideje az esetleges újrapróbálkozási kísérletekkel együtt. Ha nincs megadva, az, hogy mennyi ideig a kérelem megengedett nincs korlátozva. Más szóval a kérelem előfordulhat, hogy lefagy. |
| ServerTimeout | None | Kérés időtúllépési kerete a kiszolgálón (egész másodpercre kerekített érték). Ha nincs megadva, a kiszolgálóhoz érkező összes kérés esetében az alapértelmezett értéket fogja használni. Általában érdemes üresen hagyni ezt az értéket, hogy a kiszolgáló az alapértelmezett beállítást használja. | 
| LocationMode | None | Ha a tárfiókot az írásvédett georedundáns tároló (RA-GRS) replikációs beállítással hozza létre, a hely módja határozza meg, hogy melyik hely kapja meg a kérést. A **PrimaryThenSecondary** érték esetében például a kérések először mindig az elsődleges helyre érkeznek. Ha a kérés sikertelen, ezután a másodlagos hely kapja meg azt. |
| RetryPolicy | ExponentialPolicy | Az egyes beállítások részleteit az alábbiakban tekintheti meg. |

**Exponenciális szabályzat** 

| **Beállítás** | **Alapértelmezett érték** | **Jelentés** |
| --- | --- | --- |
| maxAttempt | 3 | Az újrapróbálkozási kísérletek száma. |
| deltaBackoff | 4 másodperc | Az újrapróbálkozások közötti visszatartási időköz. Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét (egy véletlenszerű elemet belevéve) fogja használni a rendszer. |
| MinBackoff | 3 másodperc | Ez az érték hozzáadódik a deltaBackoff alapján kiszámított minden újrapróbálkozási időközhöz. Ez az érték nem módosítható.
| MaxBackoff | 120 másodperc | A MaxBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke. Ez az érték nem módosítható. |

**Lineáris szabályzat**

| **Beállítás** | **Alapértelmezett érték** | **Jelentés** |
| --- | --- | --- |
| maxAttempt | 3 | Az újrapróbálkozási kísérletek száma. |
| deltaBackoff | 30 másodperc | Az újrapróbálkozások közötti visszatartási időköz. |

### <a name="retry-usage-guidance"></a>Újrapróbálkozásokra vonatkozó útmutató
Ügyeljen a következőkre, amikor a Storage-ügyfél API-ján keresztül próbálja elérni az Azure Storage-szolgáltatásokat:

* Használja a Microsoft.WindowsAzure.Storage.RetryPolicies névtérben található beépített újrapróbálkozási szabályzatokat, amennyiben azok megfelelnek az elvárásainak. A legtöbb esetben ezek a szabályzatok is elégségesek.
* Az **ExponentialRetry** szabályzatot kötegelt műveletek, háttérfeladatok vagy nem interaktív forgatókönyvek esetében használja. Ezekben az esetekben általában több idő áll rendelkezésre megvárni, amíg a szolgáltatás helyreáll, ami növeli a művelet sikeres teljesítésének esélyét.
* Érdemes megadni a **RequestOptions** paraméter **MaximumExecutionTime** tulajdonságát a teljes végrehajtási idő korlátozásához, de az időtúllépési érték megadásakor vegye figyelembe a művelet típusát és méretét.
* Ha egyéni újrapróbálkozást implementál, ne hozzon létre burkolókat a Storage-ügyfél osztályai körül. Inkább meglévő szabályzatokat bővítsen ki az **IExtendedRetryPolicy** felületen keresztül.
* Amennyiben írásvédett georedundáns tárolót (RA-GRS) használ, a **LocationMode** segítségével beállíthatja, hogy az elsődleges hozzáférés sikertelensége esetén az újrapróbálkozási kísérletek a tároló másodlagos írásvédett példányához férjenek hozzá. Ebben az esetben azonban meg kell győződnie arról, hogy az alkalmazás esetlegesen elavult adatokkal is képes sikeresen működni (ha az elsődleges tároló replikálása még nem fejeződött be).

A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez. Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.  

| **Környezet** | **Mintacél E2E<br />max. késleltetése** | **Újrapróbálkozási szabályzat** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felület<br />vagy előtér |2 másodperc |Lineáris |maxAttempt<br />deltaBackoff |3<br />500 ms |1. kísérlet – 500 ms késleltetés<br />2. kísérlet – 500 ms késleltetés<br />3. kísérlet – 500 ms késleltetés |
| Háttér<br />vagy kötegelt |30 másodperc |Exponenciális |maxAttempt<br />deltaBackoff |5<br />4 másodperc |1. kísérlet – kb. 3 mp. késleltetés<br />2. kísérlet – kb. 7 mp. késleltetés<br />3. kísérlet – kb. 15 mp. késleltetés |

### <a name="telemetry"></a>Telemetria
Az újrapróbálkozási kísérletek a **TraceSource** osztályban lesznek naplózva. Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia. Használja a **TextWriterTraceListener** vagy az **XmlWriterTraceListener** osztályt az adatok a naplófájlba írásához. Az **EventLogTraceListener** osztállyal a Windows eseménynaplóba írhat, míg az **EventProviderTraceListener** osztállyal a nyomkövetési adatokat rögzítheti az ETW alrendszerben. Beállíthatja, hogy a puffer ürítése automatikusan megtörténjen, valamint a naplózott események részletességét is (például: Hiba, Figyelmeztetés, Tájékoztató és Részletes). További információt a [.NET-es Storage-ügyfélkódtárral történő ügyféloldali naplózást](http://msdn.microsoft.com/library/azure/dn782839.aspx) ismertető szakaszban találhat.

A műveletek kaphatnak egy **OperationContext**-példányt, amely közzétesz egy **Retrying** eseményt, amelyhez egyéni telemetrialogika csatolható. További információ: [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).

### <a name="examples"></a>Példák
A következő mintakód azt mutatja be, hogyan lehet két **TableRequestOptions**-példányt létrehozni eltérő újrapróbálkozási beállításokkal. Az egyik az interaktív, a másik pedig a háttérkérések kezelésére szolgál. A példa ezután az ügyfélre alkalmazza ezt a két újrapróbálkozási szabályzatot, így azok minden kérésre vonatkozni fognak, ezenkívül az interaktív stratégiát egy adott kéréshez köti, így az felülbírálja az ügyfél alapértelmezett beállításait.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>További információ
* [Az Azure Storage ügyféloldali kódtárhoz javasolt újrapróbálkozási szabályzatok](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [Storage ügyféloldali kódtár 2.0 – újrapróbálkozási szabályzatok implementálása](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a>Általános REST- és újrapróbálkozási irányelvek
Vegye figyelembe a következőket, amikor az Azure-t vagy külső szolgáltatást próbál elérni:

* Alkalmazzon rendszerszintű megközelítést az újrapróbálkozások kezelésére, például egy újrafelhasználható kód formájában, hogy minden ügyfél és megoldás esetében ugyanaz a módszertan érvényesüljön.
* Érdemes lehet egy újrapróbálkozási keretrendszert használni (például az átmenetihiba-kezelési alkalmazásblokkot) az újrapróbálkozások kezelésére, ha a célszolgáltatás vagy -ügyfél nem rendelkezik beépített újrapróbálkozási mechanizmussal. Így konzisztens újrapróbálkozási viselkedés valósítható meg, és megfelelő alapértelmezett újrapróbálkozási stratégiát tehet elérhetővé a célszolgáltatás számára. Ugyanakkor szükség lehet egyéni újrapróbálkozási kód kialakítására a nem szabványos viselkedésű szolgáltatások esetében, amelyek nem kivételek alapján állapítják meg az átmeneti hibákat, illetve ha **Retry-Response** válasszal kívánja kezelni az újrapróbálkozási viselkedést.
* Az átmeneti hibák észlelési logikája a REST-hívások végrehajtására ténylegesen használt ügyfél-API-tól függ. Egyes ügyfelek, például az újabb **HttpClient** osztály, nem váltanak ki kivételeket, ha a kérés befejeződik, de nem sikert jelző HTTP-állapotkódot ad vissza. Ez növeli a teljesítményt, de megakadályozza az átmenetihiba-kezelési alkalmazásblokk használatát. Ebben az esetben a hívást olyan kóddal kell beburkolni a REST API számára, amely kivételeket vált ki a nem sikert jelző HTTP-állapotkódok esetén, amelyet aztán feldolgozhat a blokk. Ezenkívül használhat egy másik mechanizmust is az újrapróbálkozások kezelésére.
* A szolgáltatás által visszaadott HTTP-állapotkód segíthet megállapítani, hogy a hiba átmeneti jellegű-e. Lehet, hogy az állapotkód vagy az azzal egyenértékű kivételtípus megismeréséhez meg kell vizsgálnia az ügyfél vagy az újrapróbálkozási keretrendszer által előállított kivételeket. A következő HTTP-kódok általában azt jelzik, hogy érdemes újrapróbálkozni:
  * 408 Kérés időtúllépése
  * 429 túl sok kérelem
  * 500 Belső kiszolgálóhiba
  * 502 Hibás átjáró
  * 503 A szolgáltatás nem érhető el
  * 504 Időtúllépés az átjárón
* Amennyiben kivételekre épül az újrapróbálkozási logikája, a következők általában a csatlakozást meghiúsító átmeneti hibát jeleznek:
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* Amennyiben egy szolgáltatás állapota „Nem érhető el”, előfordulhat, hogy a szolgáltatás a megfelelő késleltetést jelzi, mielőtt újrapróbálkozna a **Retry-After** válaszfejlécben vagy egy másik egyéni fejlécben. A szolgáltatások további információkat küldhetnek egyéni fejlécként, vagy a válasz tartalmába beágyazva. Az átmenetihiba-kezelési alkalmazásblokk nem képes használni sem a szabványos, sem pedig az egyéni „retry-after”-fejléceket.
* Ne próbálkozzon újra, ha az állapotkód ügyfélhibát jelez (4xx-es tartomány). E szabály alól csak a 408-as Kérés időtúllépése képez kivételt.
* Tesztelje le alaposan az újrapróbálkozási stratégiáit és mechanizmusait különböző feltételek mellett, például különböző hálózati állapotok és rendszerterhelések esetén.

### <a name="retry-strategies"></a>Újrapróbálkozási stratégiák
A következők az újrapróbálkozási stratégiák időközeinek jellemző típusai:

* **Az exponenciális**. A megadott számú ki megközelítés véletlenszerű exponenciális biztonsági használatával az intervallum újrapróbálkozások között, az újrapróbálkozásokat végző újrapróbálkozási házirendje. Példa:

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* **Növekményes**. A megadott számú újrapróbálkozások és a próbálkozások növekményes időközét újrapróbálkozási stratégiát. Példa:

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* **LinearRetry**. Egy megadott időpontban időköze újrapróbálkozások között, az újrapróbálkozásokat megadott számú végző újrapróbálkozási házirendje. Példa:

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a>Átmeneti hibák kezelése a Polly használatával
[Polly] [ polly] egy függvénytár programozottan kezelni az újrapróbálkozások és [áramköri megszakító] [ circuit-breaker] stratégiák. A Polly projekt a [.NET Foundation][dotnet-foundation] keretében érhető el. A Polly olyan szolgáltatások számára kínál alternatívát, amelyekben az ügyfél nem támogatja natív módon az újrapróbálkozásokat. Megszünteti az olyan egyéni újrapróbálkozási kódok írásának szükségét, amelyeket egyébként nehéz lenne megfelelően implementálni. A Polly ezenkívül módot ad a hibák nyomon követésére, így naplózhatóvá teszi az újrapróbálkozásokat.


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a>További információ
* [Kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency)
* [Adatpontok – EF Core 1.1](https://msdn.microsoft.com/magazine/mt745093.aspx)


